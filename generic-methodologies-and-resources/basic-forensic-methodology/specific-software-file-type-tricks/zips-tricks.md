# ZIPs tricks

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Command-line alati** za upravljanje **zip fajlovima** su neophodni za dijagnostikovanje, popravku i probijanje zip fajlova. Evo nekih ključnih alata:

* **`unzip`**: Otkriva zašto se zip fajl možda ne može dekompresovati.
* **`zipdetails -v`**: Pruža detaljnu analizu polja formata zip fajla.
* **`zipinfo`**: Lista sadržaj zip fajla bez ekstrakcije.
* **`zip -F input.zip --out output.zip`** i **`zip -FF input.zip --out output.zip`**: Pokušajte da popravite oštećene zip fajlove.
* [**fcrackzip**](https://github.com/hyc/fcrackzip): Alat za brute-force probijanje lozinki zip fajlova, efikasan za lozinke do oko 7 karaktera.

Specifikacija [Zip formata fajla](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) pruža sveobuhvatne detalje o strukturi i standardima zip fajlova.

Važno je napomenuti da zip fajlovi zaštićeni lozinkom **ne šifruju imena fajlova ili veličine fajlova** unutar sebe, što je sigurnosna slabost koju ne dele RAR ili 7z fajlovi koji šifruju ove informacije. Osim toga, zip fajlovi koji su šifrovani starijom metodom ZipCrypto su ranjivi na **napad sa poznatim tekstom** ako je dostupna nešifrovana kopija komprimiranog fajla. Ovaj napad koristi poznati sadržaj za probijanje lozinke zip fajla, ranjivost koja je detaljno objašnjena u [HackThis-ovom članku](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) i dalje objašnjena u [ovom naučnom radu](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf). Međutim, zip fajlovi koji su obezbeđeni **AES-256** šifrovanjem su imuni na ovaj napad sa poznatim tekstom, što pokazuje važnost izbora sigurnih metoda šifrovanja za osetljive podatke.

## Reference

* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
