# macOS Bundles

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovne Informacije

Paketi u macOS-u služe kao kontejneri za različite resurse uključujući aplikacije, biblioteke i druge neophodne fajlove, čineći ih jedinstvenim objektima u Finder-u, poput poznatih `*.app` fajlova. Najčešće korišćeni paket je `.app` paket, mada su prisutni i drugi tipovi poput `.framework`, `.systemextension` i `.kext`.

### Osnovni Sastojci Paketa

Unutar paketa, posebno unutar direktorijuma `<aplikacija>.app/Contents/`, smešteni su različiti važni resursi:

* **\_CodeSignature**: Ovaj direktorijum čuva detalje potpisivanja koda koji su vitalni za proveru integriteta aplikacije. Možete pregledati informacije o potpisivanju koda koristeći komande poput: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Sadrži izvršni binarni fajl aplikacije koji se pokreće prilikom interakcije korisnika.
* **Resources**: Repozitorijum za komponente korisničkog interfejsa aplikacije uključujući slike, dokumente i opise interfejsa (nib/xib fajlovi).
* **Info.plist**: Deluje kao glavni konfiguracioni fajl aplikacije, od suštinskog značaja za sistem da prepozna i interaguje sa aplikacijom na odgovarajući način.

#### Važni Ključevi u Info.plist

Fajl `Info.plist` je osnova za konfiguraciju aplikacije, sadrži ključeve poput:

* **CFBundleExecutable**: Specificira ime glavnog izvršnog fajla smeštenog u direktorijumu `Contents/MacOS`.
* **CFBundleIdentifier**: Pruža globalni identifikator za aplikaciju, široko korišćen od strane macOS-a za upravljanje aplikacijama.
* **LSMinimumSystemVersion**: Označava minimalnu verziju macOS-a potrebnu za pokretanje aplikacije.

### Istraživanje Paketa

Za istraživanje sadržaja paketa, poput `Safari.app`, može se koristiti sledeća komanda: `bash ls -lR /Applications/Safari.app/Contents`

Ovo istraživanje otkriva direktorijume poput `_CodeSignature`, `MacOS`, `Resources`, i fajlove poput `Info.plist`, svaki sa jedinstvenom svrhom od osiguravanja aplikacije do definisanja njenog korisničkog interfejsa i operativnih parametara.

#### Dodatni Direktorijumi Paketa

Pored uobičajenih direktorijuma, paketi mogu takođe sadržati:

* **Frameworks**: Sadrži uvezene okvire korišćene od strane aplikacije. Okviri su poput dylibs sa dodatnim resursima.
* **PlugIns**: Direktorijum za dodatke i ekstenzije koje poboljšavaju mogućnosti aplikacije.
* **XPCServices**: Drži XPC servise korišćene od strane aplikacije za komunikaciju van procesa.

Ova struktura osigurava da su svi neophodni komponenti zatvoreni unutar paketa, olakšavajući modularno i sigurno okruženje aplikacije.

Za detaljnije informacije o ključevima `Info.plist` i njihovim značenjima, Apple-ova dokumentacija za developere pruža obimne resurse: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
