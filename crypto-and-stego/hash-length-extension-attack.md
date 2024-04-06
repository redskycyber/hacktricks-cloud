# Hash Length Extension Attack

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Rezime napada

Zamislite server koji **potpisuje** neke **podatke** tako što **dodaje** tajni ključ na neke poznate čiste tekstualne podatke, a zatim hešira te podatke. Ako znate:

* **Dužinu tajnog ključa** (ovo se takođe može probiti iz datog opsega dužine)
* **Čiste tekstualne podatke**
* **Algoritam (i da je ranjiv na ovaj napad)**
* **Padding je poznat**
* Obično se koristi podrazumevani, pa ako su ispunjena i prethodna 3 zahteva, ovo takođe važi
* Padding varira u zavisnosti od dužine tajnog ključa+podataka, zbog čega je potrebna dužina tajnog ključa

Tada je moguće da **napadač** doda **podatke** i **generiše** validan **potpis** za **prethodne podatke + dodate podatke**.

### Kako?

Osnovno ranjivi algoritmi generišu hešove tako što prvo **heširaju blok podataka**, a zatim, **iz** prethodno **kreiranog** heša (stanja), **dodaju sledeći blok podataka** i **heširaju ga**.

Zamislite da je tajni ključ "tajna" i podaci su "podaci", MD5 od "tajnapodaci" je 6036708eba0d11f6ef52ad44e8b74d5b.\
Ako napadač želi da doda string "dodatak" može:

* Generisati MD5 od 64 "A"-a
* Promeniti stanje prethodno inicijalizovanog heša na 6036708eba0d11f6ef52ad44e8b74d5b
* Dodati string "dodatak"
* Završiti heširanje i rezultujući heš će biti **validan za "tajna" + "podaci" + "padding" + "dodatak"**

### **Alat**

{% embed url="https://github.com/iagox86/hash_extender" %}

### Reference

Ovaj napad je dobro objašnjen na [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
