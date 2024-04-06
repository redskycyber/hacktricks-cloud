# Stealing Sensitive Information Disclosure from a Web

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Ako u nekom trenutku pronađete **veb stranicu koja vam prikazuje osetljive informacije na osnovu vaše sesije**: Možda reflektuje kolačiće, ili prikazuje podatke o kreditnoj kartici ili bilo koje druge osetljive informacije, možete pokušati da ih ukradete.\
Ovde vam predstavljam glavne načine na koje možete pokušati da to postignete:

* [**CORS zaobilaženje**](../pentesting-web/cors-bypass.md): Ako možete zaobići CORS zaglavlja, moći ćete da ukradete informacije izvršavajući Ajax zahtev za zlonamernu stranicu.
* [**XSS**](../pentesting-web/xss-cross-site-scripting/): Ako pronađete XSS ranjivost na stranici, možda ćete moći da je zloupotrebite kako biste ukrali informacije.
* [**Dangling Markup**](../pentesting-web/dangling-markup-html-scriptless-injection/): Ako ne možete ubaciti XSS oznake, i dalje možete pokušati da ukradete informacije koristeći druge redovne HTML oznake.
* [**Clickjaking**](../pentesting-web/clickjacking.md): Ako nema zaštite protiv ovog napada, možda ćete moći da prevarite korisnika da vam pošalje osetljive podatke (primer [ovde](https://medium.com/bugbountywriteup/apache-example-servlet-leads-to-61a2720cac20)).

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
