# macOS Dirty NIB

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Za dalje detalje o tehnici pogledajte originalni post na:** [**https://blog.xpnsec.com/dirtynib/\*\***](https://blog.xpnsec.com/dirtynib/)**.** Evo sažetka:

NIB fajlovi, deo Apple-ovog razvojnog ekosistema, služe za definisanje **UI elemenata** i njihovih interakcija u aplikacijama. Obuhvataju serijalizovane objekte kao što su prozori i dugmad, i učitavaju se pri izvršavanju. Uprkos njihovoj trenutnoj upotrebi, Apple sada preporučuje Storyboards za sveobuhvatniju vizualizaciju UI toka.

### Bezbednosne brige sa NIB fajlovima

Važno je napomenuti da **NIB fajlovi mogu predstavljati bezbednosni rizik**. Imaju potencijal da **izvršavaju proizvoljne komande**, a izmene NIB fajlova unutar aplikacije ne sprečavaju Gatekeeper da izvrši aplikaciju, što predstavlja značajnu pretnju.

### Proces zloupotrebe prljavog NIB-a

#### Kreiranje i podešavanje NIB fajla

1. **Početno podešavanje**:

* Kreirajte novi NIB fajl koristeći XCode.
* Dodajte objekat na interfejs, postavljajući mu klasu na `NSAppleScript`.
* Podesite početno svojstvo `source` putem User Defined Runtime Attributes.

2. **Kod za izvršavanje**:

* Podešavanje omogućava pokretanje AppleScript-a po potrebi.
* Integrišite dugme za aktiviranje objekta `Apple Script`, posebno pokretanje selektora `executeAndReturnError:`.

3. **Testiranje**:

* Jednostavan AppleScript za testiranje:

```bash
set theDialogText to "PWND"
display dialog theDialogText
```

* Testirajte pokretanjem u XCode debuggeru i klikom na dugme.

#### Ciljanje aplikacije (Primer: Pages)

1. **Priprema**:

* Kopirajte ciljanu aplikaciju (npr. Pages) u poseban direktorijum (npr. `/tmp/`).
* Pokrenite aplikaciju da zaobiđete probleme sa Gatekeeper-om i keširajte je.

2. **Pisanje preko NIB fajla**:

* Zamenite postojeći NIB fajl (npr. About Panel NIB) sa izrađenim DirtyNIB fajlom.

3. **Izvršavanje**:

* Pokrenite izvršavanje interakcijom sa aplikacijom (npr. izborom stavke menija `About`).

#### Dokaz o konceptu: Pristupanje korisničkim podacima

* Izmenite AppleScript da pristupite i izvučete korisničke podatke, kao što su fotografije, bez pristanka korisnika.

### Primer koda: Zlonamerni .xib fajl

* Pristupite i pregledajte [**primer zlonamernog .xib fajla**](https://gist.github.com/xpn/16bfbe5a3f64fedfcc1822d0562636b4) koji demonstrira izvršavanje proizvoljnog koda.

### Adresiranje ograničenja pokretanja

* Ograničenja pokretanja sprečavaju izvršavanje aplikacije sa neočekivanih lokacija (npr. `/tmp`).
* Moguće je identifikovati aplikacije koje nisu zaštićene ograničenjima pokretanja i ciljati ih za ubacivanje NIB fajla.

### Dodatne macOS zaštite

Od macOS Sonoma verzije nadalje, modifikacije unutar App bundle-ova su ograničene. Međutim, ranije metode su uključivale:

1. Kopiranje aplikacije na drugu lokaciju (npr. `/tmp/`).
2. Preimenovanje direktorijuma unutar App bundle-a da bi se zaobišle početne zaštite.
3. Nakon pokretanja aplikacije da se registruje kod Gatekeeper-a, modifikovanje App bundle-a (npr. zamena MainMenu.nib sa Dirty.nib).
4. Vraćanje preimenovanih direktorijuma i ponovno pokretanje aplikacije da bi se izvršio ubačeni NIB fajl.

**Napomena**: Nedavne macOS nadogradnje su umanjile ovu eksploataciju sprečavanjem modifikacija fajlova unutar App bundle-a nakon keširanja od strane Gatekeeper-a, čime je eksploatacija postala neefikasna.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
