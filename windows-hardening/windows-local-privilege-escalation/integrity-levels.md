# Integrity Levels

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Nivoi integriteta

U Windows Vista i kasnijim verzijama, svi zaštićeni objekti imaju oznaku **nivoa integriteta**. Ova postavka uglavnom dodeljuje "srednji" nivo integriteta datotekama i ključevima registra, osim određenih fascikli i datoteka na koje Internet Explorer 7 može pisati na niskom nivou integriteta. Podrazumevano ponašanje je da procesi pokrenuti od strane standardnih korisnika imaju srednji nivo integriteta, dok usluge obično rade na nivou sistema. Visok nivo integriteta štiti korenski direktorijum.

Ključno pravilo je da objekte ne mogu menjati procesi sa nižim nivoom integriteta od nivoa objekta. Nivoi integriteta su:

* **Nepoveren**: Ovaj nivo je za procese sa anonimnim prijavama. %%%Primer: Chrome%%%
* **Nizak**: Pretežno za internet interakcije, posebno u Internet Explorer-ovom Protected Mode-u, utiče na povezane datoteke i procese, kao i određene fascikle poput **Temporary Internet Folder**-a. Procesi sa niskim nivoom integriteta suočavaju se sa značajnim ograničenjima, uključujući nemogućnost pisanja u registar i ograničen pristup korisničkom profilu.
* **Srednji**: Podrazumevani nivo za većinu aktivnosti, dodeljen standardnim korisnicima i objektima bez posebnih nivoa integriteta. Čak i članovi grupe Administratori rade na ovom nivou podrazumevano.
* **Visok**: Rezervisan za administratore, omogućava im da menjaju objekte na nižim nivoima integriteta, uključujući one na visokom nivou integriteta.
* **Sistem**: Najviši operativni nivo za Windows kernel i osnovne usluge, nedostupan čak i administratorima, obezbeđujući zaštitu vitalnih sistemskih funkcija.
* **Instalater**: Jedinstven nivo koji stoji iznad svih ostalih, omogućava objektima na ovom nivou da deinstaliraju bilo koji drugi objekat.

Možete dobiti nivo integriteta procesa koristeći **Process Explorer** iz **Sysinternals**, pristupajući **svojstvima** procesa i pregledanjem kartice "**Security**":

![](<../../.gitbook/assets/image (318).png>)

Takođe možete dobiti svoj **trenutni nivo integriteta** koristeći `whoami /groups`

![](<../../.gitbook/assets/image (319).png>)

### Nivoi integriteta u fajl-sistemu

Objekt unutar fajl-sistema može zahtevati **minimalni nivo integriteta** i ako proces nema ovaj nivo integriteta, neće moći da interaguje sa njim.\
Na primer, hajde da **kreiramo običnu datoteku iz konzole običnog korisnika i proverimo dozvole**:

```
echo asd >asd.txt
icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
```

Sada ćemo dodeliti minimalni nivo integriteta **Visok** datoteci. Ovo **mora biti urađeno iz konzole** koja se pokreće kao **administrator**, jer će **obična konzola** raditi sa nivoom integriteta Srednji i **neće biti dozvoljeno** dodeljivanje nivoa integriteta Visok objektu:

```
icacls asd.txt /setintegritylevel(oi)(ci) High
processed file: asd.txt
Successfully processed 1 files; Failed processing 0 files

C:\Users\Public>icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
Mandatory Label\High Mandatory Level:(NW)
```

Ovde stvari postaju interesantne. Možete videti da korisnik `DESKTOP-IDJHTKP\user` ima **PUNE privilegije** nad fajlom (zaista, ovaj korisnik je kreirao fajl), međutim, zbog minimalnog nivoa integriteta koji je implementiran, neće moći više da menja fajl osim ako radi unutar visokog nivoa integriteta (napomena: moći će da ga čita):

```
echo 1234 > asd.txt
Access is denied.

del asd.txt
C:\Users\Public\asd.txt
Access is denied.
```

{% hint style="info" %}
**Dakle, kada datoteka ima minimalni nivo integriteta, da biste je izmenili, morate pokrenuti bar na tom nivou integriteta.**
{% endhint %}

### Nivoi integriteta u binarnim fajlovima

Napravio sam kopiju `cmd.exe` fajla pod nazivom `cmd-low.exe` u `C:\Windows\System32` direktorijumu i postavio mu **nivo integriteta na niski nivo iz administratorske konzole:**

```
icacls C:\Windows\System32\cmd-low.exe
C:\Windows\System32\cmd-low.exe NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APP PACKAGES:(I)(RX)
Mandatory Label\Low Mandatory Level:(NW)
```

Sada, kada pokrenem `cmd-low.exe`, on će **raditi sa niskim nivoom integriteta** umesto srednjeg:

![](<../../.gitbook/assets/image (320).png>)

Za radoznale ljude, ako dodelite visok nivo integriteta binarnom fajlu (`icacls C:\Windows\System32\cmd-high.exe /setintegritylevel high`), on neće automatski raditi sa visokim nivoom integriteta (ako ga pokrenete iz procesa sa srednjim nivoom integriteta - po defaultu, radiće sa srednjim nivoom integriteta).

### Nivoi Integriteta u Procesima

Nisu svi fajlovi i folderi dodeljeni minimalni nivo integriteta, **ali svi procesi rade sa određenim nivoom integriteta**. I slično kao što se desilo sa fajl-sistemom, **ako proces želi da piše unutar drugog procesa, mora imati barem isti nivo integriteta**. To znači da proces sa niskim nivoom integriteta ne može otvoriti ručku sa punim pristupom ka procesu sa srednjim nivoom integriteta.

Zbog ograničenja navedenih u ovoj i prethodnoj sekciji, sa aspekta bezbednosti, uvek je **preporučljivo pokretati proces sa što nižim nivoom integriteta**.

<details>

<summary><strong>Naučite hakovanje AWS-a od početka do naprednog nivoa sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
