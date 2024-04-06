# Infrared

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Kako radi infracrveno <a href="#kako-radi-infracrveno-port" id="kako-radi-infracrveno-port"></a>

**Infracrveno svetlo je nevidljivo ljudima**. Talasna dužina infracrvenog svetla je od **0,7 do 1000 mikrona**. Kućni daljinski upravljači koriste infracrveni signal za prenos podataka i rade u opsegu talasnih dužina od 0,75 do 1,4 mikrona. Mikrokontroler u daljinskom upravljaču čini da infracrveni LED treperi sa određenom frekvencijom, pretvarajući digitalni signal u infracrveni signal.

Za prijem infracrvenih signala koristi se **fotoprijemnik**. On **pretvara infracrveno svetlo u naponske impulse**, koji su već **digitalni signali**. Obično, unutar prijemnika se nalazi **filter tamnog svetla**, koji propušta **samo željenu talasnu dužinu** i odbacuje šum.

### Različiti infracrveni protokoli <a href="#razliciti-infracrveni-protokoli" id="razliciti-infracrveni-protokoli"></a>

Infracrveni protokoli se razlikuju u 3 faktora:

* kodiranje bitova
* struktura podataka
* nosačka frekvencija - često u opsegu od 36 do 38 kHz

#### Načini kodiranja bitova <a href="#nacini-kodiranja-bitova" id="nacini-kodiranja-bitova"></a>

**1. Kodiranje udaljenosti impulsa**

Bitovi se kodiraju modulacijom trajanja prostora između impulsa. Širina samog impulsa je konstantna.

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

**2. Kodiranje širine impulsa**

Bitovi se kodiraju modulacijom širine impulsa. Širina prostora nakon impulsa je konstantna.

<figure><img src="../../.gitbook/assets/image (29) (1).png" alt=""><figcaption></figcaption></figure>

**3. Kodiranje faze**

Takođe je poznato kao Manchester kodiranje. Logička vrednost je definisana polaritetom tranzicije između impulsa i prostora. "Prostor do impulsa" označava logičku "0", "impuls do prostora" označava logičku "1".

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

**4. Kombinacija prethodnih i drugih egzotičnih**

{% hint style="info" %}
Postoje infracrveni protokoli koji **pokušavaju postati univerzalni** za nekoliko vrsta uređaja. Najpoznatiji su RC5 i NEC. Nažalost, najpoznatiji **ne znači najčešći**. U mom okruženju sam se susreo samo sa dva NEC daljinska upravljača i nijednim RC5.

Proizvođači vole koristiti svoje jedinstvene infracrvene protokole, čak i unutar istog opsega uređaja (na primer, TV-bokseva). Zbog toga, daljinski upravljači različitih kompanija i ponekad različitih modela iste kompanije, nisu u mogućnosti da rade sa drugim uređajima iste vrste.
{% endhint %}

### Istraživanje infracrvenog signala

Najpouzdaniji način da vidite kako izgleda infracrveni signal daljinskog upravljača je korišćenje osciloskopa. On ne demoduliše ili invertuje primljeni signal, već ga prikazuje "kakav jeste". Ovo je korisno za testiranje i otklanjanje grešaka. Pokazaću očekivani signal na primeru NEC infracrvenog protokola.

<figure><img src="../../.gitbook/assets/image (18) (2).png" alt=""><figcaption></figcaption></figure>

Obično, na početku kodiranog paketa postoji preambula. To omogućava prijemniku da odredi nivo pojačanja i pozadine. Takođe postoje protokoli bez preambule, na primer, Sharp.

Zatim se prenose podaci. Struktura, preambula i način kodiranja bitova određeni su specifičnim protokolom.

**NEC infracrveni protokol** sadrži kratku komandu i ponavljajući kod, koji se šalje dok je dugme pritisnuto. I komanda i ponavljajući kod imaju istu preambulu na početku.

NEC **komanda**, pored preambule, sastoji se od bajta adrese i bajta broja komande, preko kojih uređaj razume šta treba da se izvrši. Bajtovi adrese i broja komande su duplicirani sa inverznim vrednostima, kako bi se proverila celovitost prenosa. Na kraju komande postoji dodatni stop bit.

**Ponavljajući kod** ima "1" nakon preambule, što je stop bit.

Za logičke vrednosti "0" i "1" NEC koristi kodiranje udaljenosti impulsa: prvo se prenosi impulz, nakon čega sledi pauza, čija dužina određuje vrednost bita.

### Klima uređaji

Za razliku od drugih daljinskih upravljača, **klima uređaji ne prenose samo kod pritisnutog dugmeta**. Oni takođe **prenose sve informacije** kada se dugme pritisne kako bi se osiguralo da su **klima uređaj i daljinski upravljač sinhronizovani**.\
Ovo će sprečiti da se mašina podešena na 20ºC poveća na 21ºC sa jednim daljinskim upravljačem, a zatim kada se drugi daljinski upravljač, koji još uvek ima temperaturu od 20ºC, koristi za dalje povećanje temperature, "poveća" na 21ºC (a ne na 22ºC misleći da je na 21ºC).

### Napadi

Možete napasti infracrveno pomoću Flipper Zero uređaja:

{% content-ref url="flipper-zero/fz-infrared.md" %}
[fz-infrared.md](flipper-zero/fz-infrared.md)
{% endcontent-ref %}

## Reference

* [https://blog.flipperzero.one/infrared/](https://blog.flipperzero.one/infrared/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong></summary>



</details>
