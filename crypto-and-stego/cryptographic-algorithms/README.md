# Cryptographic/Compression Algorithms

## Kriptografski/Kompresioni Algoritmi

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Identifikacija Algoritama

Ako se susretnete sa kodom **koji koristi pomeranje udesno i ulevo, ekskluzivno ili, i nekoliko aritmetičkih operacija**, vrlo je verovatno da je to implementacija **kriptografskog algoritma**. Ovde će biti prikazani neki načini **identifikacije algoritma koji se koristi bez potrebe za rekonstrukcijom svakog koraka**.

### API funkcije

**CryptDeriveKey**

Ako se koristi ova funkcija, možete pronaći koji **algoritam se koristi** proverom vrednosti drugog parametra:

![](<../../.gitbook/assets/image (375) (1) (1) (1) (1).png>)

Pogledajte ovde tabelu mogućih algoritama i njihovih dodeljenih vrednosti: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

**RtlCompressBuffer/RtlDecompressBuffer**

Komprimuje i dekomprimuje dati bafer podataka.

**CryptAcquireContext**

Iz [dokumentacije](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptacquirecontexta): Funkcija **CryptAcquireContext** se koristi za dobijanje ručke ka određenom kontejneru ključeva unutar određenog provajdera kriptografskih usluga (CSP). **Ova vraćena ručka se koristi u pozivima CryptoAPI** funkcija koje koriste izabrani CSP.

**CryptCreateHash**

Inicira heširanje niza podataka. Ako se koristi ova funkcija, možete pronaći koji **algoritam se koristi** proverom vrednosti drugog parametra:

![](<../../.gitbook/assets/image (376).png>)

Pogledajte ovde tabelu mogućih algoritama i njihovih dodeljenih vrednosti: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

### Konstante koda

Ponekad je vrlo jednostavno identifikovati algoritam zahvaljujući činjenici da mora koristiti posebnu i jedinstvenu vrednost.

![](<../../.gitbook/assets/image (370).png>)

Ako pretražite prvu konstantu na Google-u, dobićete sledeće:

![](<../../.gitbook/assets/image (371).png>)

Stoga, možete pretpostaviti da je dekompilirana funkcija **kalkulator sha256**.\
Možete pretražiti bilo koju od drugih konstanti i verovatno ćete dobiti isti rezultat.

### informacije o podacima

Ako kod nema značajne konstante, može se **učitavati informacije iz .data sekcije**.\
Možete pristupiti tim podacima, **grupisati prvi dword** i pretražiti ga na Google-u kao što smo uradili u prethodnom odeljku:

![](<../../.gitbook/assets/image (372).png>)

U ovom slučaju, ako potražite **0xA56363C6**, možete saznati da je povezano sa **tabelama AES algoritma**.

## RC4 **(Simetrična Kriptografija)**

### Karakteristike

Sastoji se od 3 glavna dela:

* **Faza inicijalizacije/**: Kreira **tabelu vrednosti od 0x00 do 0xFF** (ukupno 256 bajtova, 0x100). Ova tabela se obično naziva **Substitution Box** (ili SBox).
* **Faza mešanja**: Prolaziće kroz prethodno kreiranu tabelu (ponavljanje 0x100 iteracija, opet) modifikovanjem svake vrednosti sa **polu-slučajnim** bajtovima. Da bi se stvorili ovi polu-slučajni bajtovi, koristi se RC4 **ključ**. RC4 **ključevi** mogu biti **dugački između 1 i 256 bajtova**, mada se obično preporučuje da bude duži od 5 bajtova. Obično, RC4 ključevi su dužine 16 bajtova.
* **XOR faza**: Na kraju, plain-text ili šifrovani tekst se **XOR-uje sa vrednostima koje su prethodno kreirane**. Funkcija za šifrovanje i dešifrovanje je ista. Za to će se izvršiti **ponavljanje kroz kreiranih 256 bajtova** koliko god puta je potrebno. Ovo se obično prepoznaje u dekompiliranom kodu sa **%256 (mod 256)**.

{% hint style="info" %}
**Da biste identifikovali RC4 u disasembliranom/dekompiliranom kodu, možete proveriti da li postoje 2 petlje veličine 0x100 (sa upotrebom ključa) i zatim XOR ulaznih podataka sa 256 vrednosti koje su prethodno kreirane u 2 petlje, verovatno koristeći %256 (mod 256)**
{% endhint %}

### **Faza inicijalizacije/Substitution Box:** (Obratite pažnju na broj 256 koji se koristi kao brojač i kako je 0 upisano na svako mesto od 256 karaktera)

![](<../../.gitbook/assets/image (377).png>)

### **Faza mešanja:**

![](<../../.gitbook/assets/image (378).png>)

### **XOR faza:**

![](<../../.gitbook/assets/image (379).png>)

## **AES (Simetrična Kriptografija)**

### **Karakteristike**

* Upotreba **substitution box-ova i lookup tabela**
* Moguće je **razlikovati AES zahvaljujući upotrebi specifičnih vrednosti lookup tabela** (konstanti). _Imajte na umu da se **konstanta** može **čuvati** u binarnom **ili kreirati**_ _**dinamički**._
* **Ključ za šifrovanje** mora biti **deljiv** sa **16** (obično 32B) i obično se koristi IV od 16B.

### SBox konstante

![](<../../.gitbook/assets/image (380).png>)

## Serpent **(Simetrična Kriptografija)**

### Karakteristike

* Retko je pronaći malver koji ga koristi, ali postoje primeri (Ursnif)
* Jednostavno je odrediti da li je algoritam Serpent ili ne na osnovu njeg

## RSA **(Asimetrična kriptografija)**

### Karakteristike

* Kompleksniji od simetričnih algoritama
* Nema konstanti! (teško je odrediti prilagođenu implementaciju)
* KANAL (kripto analizator) ne može pružiti podatke o RSA jer se oslanja na konstante.

### Identifikacija pomoću poređenja

![](<../../.gitbook/assets/image (383).png>)

* U liniji 11 (levo) se nalazi `+7) >> 3`, što je isto kao u liniji 35 (desno): `+7) / 8`
* Linija 12 (levo) proverava da li je `modulus_len < 0x040`, a u liniji 36 (desno) proverava da li je `inputLen+11 > modulusLen`

## MD5 & SHA (hash)

### Karakteristike

* 3 funkcije: Init, Update, Final
* Slične inicijalizacijske funkcije

### Identifikacija

**Init**

Možete ih identifikovati proverom konstanti. Imajte na umu da sha\_init ima 1 konstantu koju MD5 nema:

![](<../../.gitbook/assets/image (385).png>)

**MD5 Transform**

Primetite upotrebu više konstanti

![](<../../.gitbook/assets/image (253) (1) (1) (1).png>)

## CRC (hash)

* Manji i efikasniji jer je njegova funkcija pronalaženje slučajnih promena u podacima
* Koristi lookup tabele (tako da možete identifikovati konstante)

### Identifikacija

Proverite **konstante lookup tabele**:

![](<../../.gitbook/assets/image (387).png>)

Algoritam za CRC hash izgleda ovako:

![](<../../.gitbook/assets/image (386).png>)

## APLib (Kompresija)

### Karakteristike

* Nema prepoznatljivih konstanti
* Možete pokušati da napišete algoritam u Pythonu i tražite slične stvari na internetu

### Identifikacija

Grafikon je prilično velik:

![](<../../.gitbook/assets/image (207) (2) (1).png>)

Proverite **3 poređenja da biste ga prepoznali**:

![](<../../.gitbook/assets/image (384).png>)

<details>

<summary><strong>Naučite hakovanje AWS-a od početnika do stručnjaka sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
