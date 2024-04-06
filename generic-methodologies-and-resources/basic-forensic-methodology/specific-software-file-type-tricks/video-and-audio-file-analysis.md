# Video and Audio file analysis

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Manipulacija audio i video fajlovima** je osnova u izazovima **CTF forenzike**, koristeći **steganografiju** i analizu metapodataka za skrivanje ili otkrivanje tajnih poruka. Alati poput [**mediainfo**](https://mediaarea.net/en/MediaInfo) i **`exiftool`** su neophodni za pregledanje metapodataka fajlova i identifikaciju vrsta sadržaja.

Za audio izazove, [**Audacity**](http://www.audacityteam.org/) se ističe kao vodeći alat za pregledanje talasnih oblika i analizu spektrograma, što je ključno za otkrivanje teksta kodiranog u audio formatu. [**Sonic Visualiser**](http://www.sonicvisualiser.org/) se visoko preporučuje za detaljnu analizu spektrograma. **Audacity** omogućava manipulaciju audio snimcima kao što su usporavanje ili obrtanje traka kako bi se otkrile skrivene poruke. [**Sox**](http://sox.sourceforge.net/), komandna linija, se odlično snalazi u konverziji i uređivanju audio fajlova.

**Manipulacija najmanje značajnim bitovima (LSB)** je česta tehnika u audio i video steganografiji, iskorišćavajući fiksne delove medijskih fajlova za skriveno ugrađivanje podataka. [**Multimon-ng**](http://tools.kali.org/wireless-attacks/multimon-ng) je koristan za dekodiranje poruka skrivenih kao **DTMF tonovi** ili **Morseov kod**.

Video izazovi često uključuju kontejnerske formate koji sadrže audio i video tokove. [**FFmpeg**](http://ffmpeg.org/) je alat za analizu i manipulaciju ovim formatima, sposoban za de-multiplexiranje i reprodukciju sadržaja. Za programere, [**ffmpy**](http://ffmpy.readthedocs.io/en/latest/examples.html) integriše mogućnosti FFmpeg-a u Python za napredne skriptabilne interakcije.

Ova paleta alata naglašava potrebnu fleksibilnost u CTF izazovima, gde učesnici moraju primeniti širok spektar tehnika analize i manipulacije kako bi otkrili skrivene podatke unutar audio i video fajlova.

### Reference

* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
