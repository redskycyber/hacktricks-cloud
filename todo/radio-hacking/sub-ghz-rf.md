# Sub-GHz RF

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Garažna vrata

Otvorivači garažnih vrata obično rade na frekvencijama u opsegu od 300-190 MHz, pri čemu su najčešće frekvencije 300 MHz, 310 MHz, 315 MHz i 390 MHz. Ovaj frekvencijski opseg se često koristi za otvorivače garažnih vrata jer je manje zagušen od drugih frekvencijskih opsega i manje je verovatno da će doživeti smetnje od drugih uređaja.

## Vrata automobila

Većina daljinskih upravljača za ključeve automobila radi na frekvencijama **315 MHz ili 433 MHz**. Ovo su obe radio frekvencije i koriste se u različitim aplikacijama. Glavna razlika između ove dve frekvencije je da 433 MHz ima veći domet od 315 MHz. To znači da je 433 MHz bolji za aplikacije koje zahtevaju veći domet, kao što je daljinsko zaključavanje vrata.\
U Evropi se obično koristi 433.92 MHz, a u SAD-u i Japanu je to 315 MHz.

## **Brute-force napad**

<figure><img src="../../.gitbook/assets/image (4) (3) (2).png" alt=""><figcaption></figcaption></figure>

Ako umesto slanja svakog koda 5 puta (slanje na ovaj način kako bi se osiguralo da prijemnik dobije signal), samo jednom pošaljete, vreme se smanjuje na 6 minuta:

<figure><img src="../../.gitbook/assets/image (1) (1) (2) (2).png" alt=""><figcaption></figcaption></figure>

i ako **uklonite period čekanja od 2 ms** između signala, vreme se može smanjiti na 3 minuta.

Osim toga, korišćenjem De Brojnovog niza (način da se smanji broj bitova potrebnih za slanje svih potencijalnih binarnih brojeva za brute force), ovo vreme se smanjuje na samo 8 sekundi:

<figure><img src="../../.gitbook/assets/image (5) (2) (3).png" alt=""><figcaption></figcaption></figure>

Primer ovog napada je implementiran na [https://github.com/samyk/opensesame](https://github.com/samyk/opensesame)

Zahtevanje **preambule će sprečiti De Brojnov niz** optimizaciju i **promenljivi kodovi će sprečiti ovaj napad** (pretpostavljajući da je kod dovoljno dug da ne može biti brute force-an).

## Napad na Sub-GHz

Da biste napali ove signale sa Flipper Zero proverite:

{% content-ref url="flipper-zero/fz-sub-ghz.md" %}
[fz-sub-ghz.md](flipper-zero/fz-sub-ghz.md)
{% endcontent-ref %}

## Zaštita od promenljivih kodova

Automatski otvarači garažnih vrata obično koriste bežični daljinski upravljač za otvaranje i zatvaranje garažnih vrata. Daljinski upravljač **šalje radio frekvencijski (RF) signal** otvaraču garažnih vrata, koji aktivira motor za otvaranje ili zatvaranje vrata.

Moguće je da neko koristi uređaj poznat kao "code grabber" da presretne RF signal i zabeleži ga za kasniju upotrebu. Ovo se naziva **replay napad**. Da bi se sprečio ovaj tip napada, mnogi moderni otvarači garažnih vrata koriste sigurniju metodu enkripcije poznatu kao **sistem promenljivih kodova**.

RF signal se obično prenosi koristeći promenljivi kod, što znači da se kod menja pri svakoj upotrebi. To otežava nekome da presretne signal i koristi ga za neovlašćeni pristup garaži.

U sistemu promenljivih kodova, daljinski upravljač i otvarač garažnih vrata imaju **deljeni algoritam** koji **generiše novi kod** svaki put kada se daljinski upravljač koristi. Otvarač garažnih vrata će odgovoriti samo na **ispravan kod**, što znači da je mnogo teže nekome da neovlašćeno pristupi garaži samo presretanjem koda.

### **Napad na izgubljenu vezu**

U osnovi, slušate dugme i **presrećete signal dok je daljinski van dometa** uređaja (na primer automobila ili garaže). Zatim se pomerite do uređaja i **koristite presretnuti kod da ga otvorite**.

### Napad potpunog blokiranja veze

Napadač može **blokirati signal blizu vozila ili prijemnika** tako da **prijemnik ne može "čuti" kod**, a kada se to dogodi, jednostavno možete **presnimiti i reprodukovati** kod kada prestanete sa blokiranjem.

Žrtva će u nekom trenutku koristiti **ključeve da zaključa automobil**, ali tada će napad imati **zabeležen dovoljan broj "zatvori vrata" kodova** koji se nadaju da bi mogli biti ponovo poslati da otvore vrata (možda će biti potrebna **promena frekvencije** jer postoje automobili koji koriste iste kodove za otvaranje i zatvaranje, ali slušaju oba komanda na različitim frekvencijama).

{% hint style="warning" %}
**Blokiranje funkcioniše**, ali je primetno jer ako **osoba koja zaključava automobil jednostavno proveri vrata** da se uveri da su zaključana, primetiće da je automobil otključan. Dodatno, ako su svesni takvih napada, mogu čak i da primete da vrata nikada nisu proizvela zvuk **zaključavanja** ili da se **svetla** automobila nisu upalila kada su pritisnuli dugme "zaključavanje".
{% endhint %}

### **Napad na presretanje koda (poznat kao 'RollJam')**

Ovo je nešto **skrivenija tehnika blokiranja**. Napadač će blokirati signal, tako da kada žrtva pokuša da zaključa vrata, to neće uspeti, ali napadač će **zabeležiti ovaj kod**. Zatim, žrtva će **ponovo pokušati da zaključa automobil** pritiskom na dugme i automobil će \*\*zabele

### Napad na ometanje zvučnog alarma

Testiranje protiv naknadno instaliranog sistema sa kodom koji se menja na automobilu, **slanje istog koda dva puta** odmah **aktivira alarm** i imobilajzer pružajući jedinstvenu priliku za **uslugu odbijanja**. Ironično, način **onemogućavanja alarma** i imobilajzera je **pritisak** na **daljinski upravljač**, pružajući napadaču mogućnost **kontinuiranog izvođenja napada odbijanja usluge**. Ili kombinujte ovaj napad sa **prethodnim da biste dobili više kodova**, jer bi žrtva želela da zaustavi napad što je pre moguće.

## Reference

* [https://www.americanradioarchives.com/what-radio-frequency-does-car-key-fobs-run-on/](https://www.americanradioarchives.com/what-radio-frequency-does-car-key-fobs-run-on/)
* [https://www.andrewmohawk.com/2016/02/05/bypassing-rolling-code-systems/](https://www.andrewmohawk.com/2016/02/05/bypassing-rolling-code-systems/)
* [https://samy.pl/defcon2015/](https://samy.pl/defcon2015/)
* [https://hackaday.io/project/164566-how-to-hack-a-car/details](https://hackaday.io/project/164566-how-to-hack-a-car/details)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje trikove hakovanja slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
