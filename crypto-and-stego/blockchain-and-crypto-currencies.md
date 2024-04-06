# Blockchain & Crypto Currencies

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### Osnovni Koncepti

* **Pametni ugovori** se definišu kao programi koji se izvršavaju na blockchain-u kada se ispune određeni uslovi, automatizujući izvršenje sporazuma bez posrednika.
* **Decentralizovane aplikacije (dApps)** se grade na pametnim ugovorima, sa korisničkim interfejsom koji je prijateljski nastrojen i transparentnim, proverljivim backend-om.
* **Tokeni & Kriptovalute** se razlikuju gde kriptovalute služe kao digitalni novac, dok tokeni predstavljaju vrednost ili vlasništvo u određenim kontekstima.
* **Utility Tokeni** omogućavaju pristup uslugama, a **Security Tokeni** označavaju vlasništvo nad imovinom.
* **DeFi** označava Decentralizovanu Finansiju, koja nudi finansijske usluge bez centralnih autoriteta.
* **DEX** i **DAO** se odnose na Decentralizovane Platforme za Razmenu i Decentralizovane Autonomne Organizacije, redom.

### Mekanizmi Konsenzusa

Mekanizmi konsenzusa obezbeđuju sigurnu i dogovorenu validaciju transakcija na blockchain-u:

* **Proof of Work (PoW)** se oslanja na računarsku snagu za verifikaciju transakcija.
* **Proof of Stake (PoS)** zahteva od validatora da poseduju određenu količinu tokena, smanjujući potrošnju energije u poređenju sa PoW-om.

### Osnove Bitkoina

#### Transakcije

Bitkoin transakcije uključuju prenos sredstava između adresa. Transakcije se validiraju putem digitalnih potpisa, obezbeđujući da samo vlasnik privatnog ključa može pokrenuti prenose.

**Ključni Elementi:**

* **Multisignature Transakcije** zahtevaju više potpisa za autorizaciju transakcije.
* Transakcije se sastoje od **ulaza** (izvor sredstava), **izlaza** (odredište), **naknade** (plaćene rudarima) i **skripti** (pravila transakcije).

#### Lightning Mreža

Cilj je poboljšati skalabilnost Bitkoina omogućavajući više transakcija unutar kanala, pri čemu se samo konačno stanje emituje na blockchain.

### Problemi Privatnosti Bitkoina

Napadi na privatnost, poput **Zajedničkog Vlasništva Ulaza** i **Detekcije Adrese za Promenu UTXO**, iskorišćavaju obrasce transakcija. Strategije poput **Miksera** i **CoinJoin-a** poboljšavaju anonimnost tako što zamagljuju veze između transakcija između korisnika.

### Anonimno Nabavljanje Bitkoina

Metode uključuju gotovinske razmene, rudarenje i korišćenje miksera. **CoinJoin** meša više transakcija kako bi otežao praćenje, dok **PayJoin** prikriva CoinJoin kao redovne transakcije za povećanu privatnost.

## Napadi na Privatnost Bitkoina

## Rezime Napada na Privatnost Bitkoina

U svetu Bitkoina, privatnost transakcija i anonimnost korisnika često su predmet zabrinutosti. Evo pojednostavljenog pregleda nekoliko uobičajenih metoda putem kojih napadači mogu ugroziti privatnost Bitkoina.

### **Pretpostavka o Zajedničkom Vlasništvu Ulaza**

Uobičajeno je da se ulazi različitih korisnika retko kombinuju u jednoj transakciji zbog složenosti. Stoga, **dve adrese ulaza u istoj transakciji često se smatraju da pripadaju istom vlasniku**.

### **Detekcija Adrese za Promenu UTXO**

UTXO, ili **Unspent Transaction Output**, mora biti u potpunosti potrošen u transakciji. Ako samo deo njega bude poslat na drugu adresu, preostali deo ide na novu adresu za promenu. Posmatrači mogu pretpostaviti da ova nova adresa pripada pošiljaocu, ugrožavajući privatnost.

#### Primer

Da bi se to izbeglo, mikseri ili korišćenje više adresa mogu pomoći u zamagljivanju vlasništva.

### **Izloženost na Društvenim Mrežama i Forumima**

Korisnici ponekad dele svoje Bitkoin adrese na mreži, što olakšava povezivanje adrese sa njenim vlasnikom.

### **Analiza Grafa Transakcija**

Transakcije se mogu vizualizovati kao grafovi, otkrivajući potencijalne veze između korisnika na osnovu toka sredstava.

### **Heuristika Nepotrebnog Ulaza (Optimalna Heuristika za Promenu)**

Ova heuristika se zasniva na analizi transakcija sa više ulaza i izlaza kako bi se pretpostavilo koji izlaz predstavlja promenu koja se vraća pošiljaocu.

#### Primer

```bash
2 btc --> 4 btc
3 btc     1 btc
```

### **Prisilno ponovno korišćenje adresa**

Napadači mogu poslati male iznose na prethodno korišćene adrese, nadajući se da će primalac te adrese kombinovati te iznose sa drugim ulazima u budućim transakcijama, čime će povezati adrese.

#### Ispravno ponašanje novčanika

Novčanici bi trebali izbegavati korišćenje novčića primljenih na već korišćenim, praznim adresama kako bi se sprečilo otkrivanje privatnosti.

### **Druge tehnike analize blokčejna**

* **Tačni iznosi plaćanja:** Transakcije bez promene verovatno su između dve adrese koje pripadaju istom korisniku.
* **Okrugli iznosi:** Okrugao iznos u transakciji sugeriše da je to plaćanje, pri čemu je izlaz koji nije okrugao verovatno promena.
* **Identifikacija novčanika:** Različiti novčanici imaju jedinstvene obrasce kreiranja transakcija, što omogućava analitičarima da identifikuju korišćeni softver i potencijalno adresu za promenu.
* **Korelacije iznosa i vremena:** Otkrivanje vremena ili iznosa transakcija može dovesti do praćenja transakcija.

### **Analiza saobraćaja**

Prateći saobraćaj na mreži, napadači mogu potencijalno povezati transakcije ili blokove sa IP adresama, ugrožavajući privatnost korisnika. Ovo je posebno tačno ako entitet ima mnogo Bitcoin čvorova, što povećava njihovu sposobnost praćenja transakcija.

### Više informacija

Za sveobuhvatan spisak napada na privatnost i odbrana, posetite [Bitcoin Privacy na Bitcoin Wiki](https://en.bitcoin.it/wiki/Privacy).

## Anonimne Bitcoin transakcije

### Načini anonimnog dobijanja Bitcoina

* **Gotovinske transakcije**: Dobijanje Bitcoina putem gotovine.
* **Alternativne gotovinske opcije**: Kupovina poklon kartica i njihova zamena za Bitcoin putem interneta.
* **Rudarenje**: Najprivatniji način za zaradu Bitcoina je rudarenje, posebno kada se radi samostalno, jer rudarski bazeni mogu znati IP adresu rudara. [Informacije o rudarskim bazenima](https://en.bitcoin.it/wiki/Pooled\_mining)
* **Krađa**: Teorijski, krađa Bitcoina može biti još jedan način anonimnog dobijanja, iako je ilegalna i nije preporučljiva.

### Usluge mešanja

Korišćenjem usluge mešanja, korisnik može **poslati Bitcoine** i dobiti **različite Bitcoine zauzvrat**, što otežava praćenje originalnog vlasnika. Međutim, ovo zahteva poverenje u uslugu da ne čuva logove i da zaista vrati Bitcoine. Alternativne opcije za mešanje uključuju Bitcoin kazina.

### CoinJoin

**CoinJoin** spaja više transakcija različitih korisnika u jednu, otežavajući proces svima koji pokušavaju da upare ulaze i izlaze. Uprkos njegovoj efikasnosti, transakcije sa jedinstvenim veličinama ulaza i izlaza i dalje mogu potencijalno biti praćene.

Primeri transakcija koje su možda koristile CoinJoin uključuju `402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a` i `85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238`.

Za više informacija, posetite [CoinJoin](https://coinjoin.io/en). Za sličnu uslugu na Ethereumu, pogledajte [Tornado Cash](https://tornado.cash), koji anonimizuje transakcije sa sredstvima od rudara.

### PayJoin

Varijanta CoinJoin-a, **PayJoin** (ili P2EP), prikriva transakciju između dve strane (na primer, kupca i trgovca) kao običnu transakciju, bez karakterističnih jednakih izlaza koje ima CoinJoin. Ovo ga čini izuzetno teškim za otkrivanje i može poništiti heuristiku o zajedničkom vlasništvu ulaza koju koriste entiteti za nadzor transakcija.

```plaintext
2 btc --> 3 btc
5 btc     4 btc
```

Transakcije poput one iznad mogu biti PayJoin, poboljšavajući privatnost dok ostaju neodvojive od standardnih bitkoin transakcija.

**Upotreba PayJoin-a može značajno poremetiti tradicionalne metode nadzora**, što je obećavajući razvoj u potrazi za transakcijskom privatnošću.

## Najbolje prakse za privatnost u kriptovalutama

### **Tehnike sinhronizacije novčanika**

Da bi se održala privatnost i sigurnost, ključno je sinhronizovati novčanike sa blokčejnom. Dve metode se ističu:

* **Puni čvor**: Preuzimanjem celokupnog blokčejna, puni čvor obezbeđuje maksimalnu privatnost. Sve transakcije ikada izvršene se čuvaju lokalno, što onemogućava protivnicima da identifikuju koje transakcije ili adrese korisniku predstavljaju interes.
* **Filtriranje blokova na strani klijenta**: Ova metoda podrazumeva kreiranje filtera za svaki blok u blokčejnu, omogućavajući novčanicima da identifikuju relevantne transakcije bez otkrivanja specifičnih interesa posmatračima na mreži. Lagani novčanici preuzimaju ove filtere, preuzimajući pune blokove samo kada se pronađe podudaranje sa adresama korisnika.

### **Korišćenje Tor-a za anonimnost**

S obzirom da Bitcoin funkcioniše na peer-to-peer mreži, preporučuje se korišćenje Tor-a kako bi se sakrila IP adresa i poboljšala privatnost prilikom interakcije sa mrežom.

### **Prevencija ponovne upotrebe adresa**

Da bi se zaštitila privatnost, važno je koristiti novu adresu za svaku transakciju. Ponovna upotreba adresa može ugroziti privatnost povezivanjem transakcija sa istim entitetom. Moderne novčanike odvraćaju od ponovne upotrebe adresa svojim dizajnom.

### **Strategije za privatnost transakcija**

* **Više transakcija**: Podela plaćanja na nekoliko transakcija može zamagliti iznos transakcije i ometati napade na privatnost.
* **Izbegavanje kusura**: Odabir transakcija koje ne zahtevaju izlaz za kusur poboljšava privatnost ometanjem metoda za otkrivanje kusura.
* **Više izlaza za kusur**: Ako izbegavanje kusura nije izvodljivo, generisanje više izlaza za kusur i dalje može poboljšati privatnost.

## **Monero: Znak anonimnosti**

Monero se bavi potrebom za apsolutnom anonimnošću u digitalnim transakcijama, postavljajući visok standard za privatnost.

## **Ethereum: Gas i transakcije**

### **Razumevanje Gasa**

Gas meri računarski napor potreban za izvršavanje operacija na Ethereumu, ceneći se u **gwei**-ima. Na primer, transakcija koja košta 2.310.000 gwei (ili 0,00231 ETH) uključuje limit gasa i osnovnu naknadu, sa napojnicom za podsticanje rudara. Korisnici mogu postaviti maksimalnu naknadu kako bi se osigurali da ne preplaćuju, a višak se vraća.

### **Izvršavanje transakcija**

Transakcije na Ethereumu uključuju pošiljaoca i primaoca, koji mogu biti korisničke ili pametne ugovorne adrese. One zahtevaju naknadu i moraju biti rudarene. Bitne informacije u transakciji uključuju primaoca, potpis pošiljaoca, vrednost, opcioni podaci, limit gasa i naknade. Važno je napomenuti da se adresa pošiljaoca izvodi iz potpisa, čime se eliminiše potreba za njom u podacima transakcije.

Ove prakse i mehanizmi su osnovni za sve one koji žele da se bave kriptovalutama uz prioritetizaciju privatnosti i sigurnosti.

### Reference

* [https://en.wikipedia.org/wiki/Proof\_of\_stake](https://en.wikipedia.org/wiki/Proof\_of\_stake)
* [https://www.mycryptopedia.com/public-key-private-key-explained/](https://www.mycryptopedia.com/public-key-private-key-explained/)
* [https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions](https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions)
* [https://ethereum.org/en/developers/docs/transactions/](https://ethereum.org/en/developers/docs/transactions/)
* [https://ethereum.org/en/developers/docs/gas/](https://ethereum.org/en/developers/docs/gas/)
* [https://en.bitcoin.it/wiki/Privacy](https://en.bitcoin.it/wiki/Privacy#Forced\_address\_reuse)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, pogledajte [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
