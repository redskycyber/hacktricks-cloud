# Radio

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## SigDigger

[**SigDigger** ](https://github.com/BatchDrake/SigDigger)je besplatan digitalni signalni analizator za GNU/Linux i macOS, dizajniran za izvlačenje informacija iz nepoznatih radio signala. Podržava različite SDR uređaje putem SoapySDR-a i omogućava prilagodljivu demodulaciju FSK, PSK i ASK signala, dekodiranje analognog videa, analizu signalnih impulsa i slušanje analognih glasovnih kanala (sve u realnom vremenu).

### Osnovna konfiguracija

Nakon instalacije, postoji nekoliko stvari koje možete razmotriti konfigurisanje.\
U podešavanjima (drugi dugme na kartici) možete odabrati **SDR uređaj** ili **odabrati datoteku** za čitanje, frekvenciju za sintezu i stopu uzorkovanja (preporučuje se do 2,56Msps ako vaš računar to podržava)\\

![](<../../.gitbook/assets/image (655) (1).png>)

U ponašanju GUI-a se preporučuje omogućavanje nekoliko stvari ako vaš računar to podržava:

![](<../../.gitbook/assets/image (465) (2).png>)

{% hint style="info" %}
Ako primetite da vaš računar ne hvata stvari, pokušajte da onemogućite OpenGL i smanjite stopu uzorkovanja.
{% endhint %}

### Upotrebe

* Samo da **uhvatite neko vreme signala i analizirate ga**, samo držite dugme "Push to capture" koliko god vam je potrebno.

![](<../../.gitbook/assets/image (631).png>)

* **Tuner** u SigDigger-u pomaže da se **uhvate bolji signali** (ali ih takođe može pogoršati). Idealno je početi sa 0 i povećavati ga sve dok ne pronađete da je **šum** koji se uvodi **veći** od **poboljšanja signala** koje vam je potrebno).

![](<../../.gitbook/assets/image (658).png>)

### Sinhronizacija sa radio kanalom

Sa [**SigDigger** ](https://github.com/BatchDrake/SigDigger)sinhronizujte se sa kanalom koji želite da čujete, konfigurišite opciju "Baseband audio preview", konfigurišite širinu opsega da biste dobili sve informacije koje se šalju, a zatim postavite Tuner na nivo pre nego što šum počne stvarno da se povećava:

![](<../../.gitbook/assets/image (389).png>)

## Interesantni trikovi

* Kada uređaj šalje nizove informacija, obično će **prvi deo biti preambula**, tako da se **ne morate brinuti** ako **ne pronađete informacije** tamo **ili ako postoje neke greške**.
* U okvirima informacija obično biste trebali **pronaći različite okvire dobro poravnate međusobno**:

![](<../../.gitbook/assets/image (660) (1).png>)

![](<../../.gitbook/assets/image (652) (1) (1).png>)

* **Nakon što povratite bitove, možda ćete morati da ih obradite na neki način**. Na primer, kodiranje u Manchesteru, gore+dole će biti 1 ili 0, a dole+gore će biti drugi. Dakle, parovi 1 i 0 (gore i dole) će biti pravi 1 ili pravi 0.
* Čak i ako signal koristi kodiranje u Manchesteru (nemoguće je pronaći više od dva 0 ili 1 uzastopno), možete **pronaći nekoliko 1 ili 0 zajedno u preambuli**!

### Otkrivanje vrste modulacije pomoću IQ

Postoje 3 načina za skladištenje informacija u signalima: Modulacija **amplitudom**, **frekvencijom** ili **fazom**.\
Ako proveravate signal, postoje različiti načini da pokušate da saznate šta se koristi za skladištenje informacija (pronađite više načina u nastavku), ali dobar način je da proverite IQ grafikon.

![](<../../.gitbook/assets/image (630).png>)

* **Otkrivanje AM-a**: Ako se na IQ grafikonu pojave na primer **2 kruga** (verovatno jedan u 0 i drugi u drugoj amplitudi), to može značiti da je ovo AM signal. To je zato što je na IQ grafikonu udaljenost između 0 i kruga amplituda signala, pa je lako vizualizovati različite amplitude koje se koriste.
* **Otkrivanje PM-a**: Kao i na prethodnoj slici, ako pronađete male krugove koji nisu povezani međusobno, verovatno znači da se koristi fazna modulacija. To je zato što je na IQ grafikonu ugao između tačke i 0,0 faza signala, pa to znači da se koristi 4 različite faze.
* Imajte na umu da ako se informacija krije u činjenici da se menja faza, a ne u samoj fazi, nećete jasno videti različite faze.
* **Otkrivanje FM-a**: IQ nema polje za identifikaciju frekvencija (udaljenost od centra je amplituda, a ugao je faza).\
  Stoga, da biste identifikovali FM, trebali biste **videti samo osnovno krug** na ovom grafikonu.\
  Osim toga, drugačija frekvencija je "predstavljena" na IQ grafikonu ubrzanjem brzine duž kruga (pa u SysDigger-u, kada izaberete signal, IQ grafikon se popunjava, ako pronađete ubrzanje ili promenu pravca u stvorenom krugu, to može značiti da je ovo FM):

## Primer AM-a

{% file src="../../.gitbook/assets/sigdigger_20220308_165547Z_2560000_433500000_float32_iq.raw" %}

### Otkrivanje AM-a

#### Provera omotača

Provera AM informacija sa [**SigDigger** ](https://github.com/BatchDrake/SigDigger)i samo gledanje **omotača** možete videti različite jasne nivoe amplitude. Korišćeni signal šalje impulse sa informacijama u AM, evo kako izgleda jedan impuls:

![](<../../.gitbook/assets/image (636).png>)

A ovo je kako deo simbola izgleda sa talasom:

![](<../../.gitbook/assets/image (650) (1).png>)

#### Provera histograma

Možete **odabrati ceo signal**

#### Sa IQ

U ovom primeru možete videti kako postoji **veliki krug**, ali i **mnogo tačaka u centru**.

![](<../../.gitbook/assets/image (640).png>)

### Dobijanje simboličke stope

#### Sa jednim simbolom

Izaberite najmanji simbol koji možete pronaći (tako da budete sigurni da je samo jedan) i proverite "Selection freq". U ovom slučaju to bi bilo 1.013kHz (tj. 1kHz).

![](<../../.gitbook/assets/image (638) (1).png>)

#### Sa grupom simbola

Takođe možete naznačiti broj simbola koje ćete izabrati i SigDigger će izračunati frekvenciju jednog simbola (što više simbola izaberete, verovatno će biti bolje). U ovom scenariju sam izabrao 10 simbola i "Selection freq" je 1.004 Khz:

![](<../../.gitbook/assets/image (635).png>)

### Dobijanje bitova

Nakon što ste otkrili da je ovo **AM modulisani** signal i **simbolička stopa** (i znajući da u ovom slučaju nešto gore znači 1, a nešto dole znači 0), vrlo je lako **dobiti bitove** kodirane u signalu. Dakle, izaberite signal sa informacijama i konfigurišite uzorkovanje i odlučivanje, a zatim pritisnite uzorak (proverite da je izabrana **Amplituda**, konfigurisana otkrivena **simbolička stopa** i izabrano **Gadner oporavak sata**):

![](<../../.gitbook/assets/image (642) (1).png>)

* **Sync to selection intervals** znači da ako ste prethodno izabrali intervale da biste pronašli simboličku stopu, ta simbolička stopa će se koristiti.
* **Manual** znači da će se koristiti naznačena simbolička stopa
* U **Fixed interval selection** naznačujete broj intervala koji treba da budu izabrani i izračunava se simbolička stopa iz toga
* **Gadner oporavak sata** obično je najbolja opcija, ali i dalje morate naznačiti neku približnu simboličku stopu.

Pritiskom na uzorak pojavljuje se ovo:

![](<../../.gitbook/assets/image (659).png>)

Sada, da biste SigDiggeru objasnili **gde je opseg** nivoa koji prenose informacije, trebate kliknuti na **niži nivo** i držati kliknutim dok ne dođete do najvećeg nivoa:

![](<../../.gitbook/assets/image (662) (1) (1) (1).png>)

Ako bi na primer postojalo **4 različita nivoa amplitude**, trebalo bi da konfigurišete **Bits per symbol na 2** i izaberete od najmanjeg do najvećeg.

Na kraju, **povećavanjem** **Zuma** i **menjanjem veličine reda** možete videti bitove (i možete sve izabrati i kopirati da biste dobili sve bitove):

![](<../../.gitbook/assets/image (649) (1).png>)

Ako signal ima više od 1 bita po simbolu (na primer 2), SigDigger **nema načina da zna koji simbol je** 00, 01, 10, 11, pa će koristiti različite **nijanse sive** za prikaz svakog (i ako kopirate bitove, koristiće **brojeve od 0 do 3**, moraćete da ih obradite).

Takođe, koristite **kodifikacije** kao što su **Manchester**, i **gore+dole** može biti **1 ili 0**, a dole+gore može biti 1 ili 0. U tim slučajevima morate **obraditi dobijene uspone (1) i padove (0)** da biste zamenili parove 01 ili 10 kao 0 ili 1.

## FM primer

{% file src="../../.gitbook/assets/sigdigger_20220308_170858Z_2560000_433500000_float32_iq.raw" %}

### Otkrivanje FM-a

#### Provera frekvencija i oblika talasa

Primer signala koji šalje informacije modulisane u FM-u:

![](<../../.gitbook/assets/image (661) (1).png>)

Na prethodnoj slici možete primetiti da se koriste **2 frekvencije**, ali ako **posmatrate** **oblik talasa**, možda nećete moći tačno identifikovati 2 različite frekvencije:

![](<../../.gitbook/assets/image (653).png>)

To je zato što sam snimio signal u obe frekvencije, pa je jedna otprilike negativna u odnosu na drugu:

![](<../../.gitbook/assets/image (656).png>)

Ako je sinhronizovana frekvencija **bliža jednoj frekvenciji nego drugoj**, lako možete videti 2 različite frekvencije:

![](<../../.gitbook/assets/image (648) (1) (1) (1).png>)

![](<../../.gitbook/assets/image (634).png>)

#### Provera histograma

Proverom histograma frekvencija signala sa informacijama lako možete videti 2 različita signala:

![](<../../.gitbook/assets/image (657).png>)

U ovom slučaju, ako proverite **histogram amplitude**, pronaći ćete **samo jednu amplitudu**, pa **ne može biti AM** (ako pronađete puno amplituda, to može biti zato što je signal gubio snagu duž kanala):

![](<../../.gitbook/assets/image (646).png>)

A ovo bi bio histogram faze (što vrlo jasno pokazuje da signal nije modulisan u fazi):

![](<../../.gitbook/assets/image (201) (2).png>)

#### Sa IQ

IQ nema polje za identifikaciju frekvencija (udaljenost od centra je amplituda, a ugao je faza).\
Stoga, da biste identifikovali FM, trebali biste **videti samo osnovno krug** na ovom grafikonu.\
Osim toga, druga frekvencija je "predstavljena" na IQ grafikonu **ubrzanjem brzine duž kruga** (pa u SysDiggeru, kada izaberete signal, IQ grafikon se popunjava, ako pronađete ubrzanje ili promenu pravca u stvorenom krugu, to bi moglo značiti da je ovo FM):

![](<../../.gitbook/assets/image (643) (1).png>)

### Dobijanje simboličke stope

Možete koristiti **istu tehniku kao u AM primeru** da biste dobili simboličku stopu kada pronađete frekvencije koje nose simbole.

### Dobijanje bitova

Možete koristiti **istu tehniku kao u AM primeru** da biste dobili bitove kada pronađete da je signal modulisan u frekvenciji i simboličku stopu.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
