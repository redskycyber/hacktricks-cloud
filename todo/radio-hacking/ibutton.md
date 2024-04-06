# iButton

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Uvod

iButton je generički naziv za elektronski identifikacioni ključ upakovan u **kovčeg u obliku novčića**. Takođe se naziva i **Dallas Touch** Memory ili kontakt memorija. Iako se često pogrešno naziva "magnetni" ključ, u stvari u njemu nema **ničega magnetnog**. U stvari, unutra je sakrivena potpuno funkcionalna **mikročip** koji radi na digitalnom protokolu.

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

### Šta je iButton? <a href="#what-is-ibutton" id="what-is-ibutton"></a>

Obično, iButton podrazumeva fizički oblik ključa i čitača - okrugli novčić sa dva kontakta. Za okvir koji ga okružuje, postoji mnogo varijacija, od najčešćeg plastičnog držača sa rupom do prstenova, privezaka, itd.

<figure><img src="../../.gitbook/assets/image (23) (2).png" alt=""><figcaption></figcaption></figure>

Kada ključ dođe do čitača, **kontakti se dodiruju** i ključ se napaja da **prepozna** svoj ID. Ponekad ključ **nije odmah pročitan** jer je **kontakt PSD interfona veći** nego što bi trebalo da bude. Zato spoljni konturi ključa i čitača ne mogu da se dodirnu. Ako je to slučaj, moraćete pritisnuti ključ preko jednog od zidova čitača.

<figure><img src="../../.gitbook/assets/image (21) (2).png" alt=""><figcaption></figcaption></figure>

### **1-Wire protokol** <a href="#id-1-wire-protocol" id="id-1-wire-protocol"></a>

Dallas ključevi razmenjuju podatke koristeći 1-wire protokol. Sa samo jednim kontaktom za prenos podataka (!!) u oba smera, od mastera do roba i obrnuto. 1-wire protokol radi prema modelu Master-Rob. U ovoj topologiji, Master uvek inicira komunikaciju, a Rob prati njegove instrukcije.

Kada ključ (Rob) kontaktira interfonski uređaj (Master), čip unutar ključa se uključuje, napajan od strane interfona, i ključ se inicijalizuje. Nakon toga, interfonski uređaj zahteva ID ključa. Sledeće, pogledaćemo ovaj proces detaljnije.

Flipper može raditi i u Master i u Rob režimu. U režimu čitanja ključa, Flipper deluje kao čitač, odnosno radi kao Master. A u režimu emulacije ključa, Flipper se pretvara u ključ, odnosno radi kao Rob.

### Dallas, Cyfral & Metakom ključevi

Za informacije o tome kako ovi ključevi funkcionišu, pogledajte stranicu [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)

### Napadi

iButton ključevi mogu biti napadnuti pomoću Flipper Zero uređaja:

{% content-ref url="flipper-zero/fz-ibutton.md" %}
[fz-ibutton.md](flipper-zero/fz-ibutton.md)
{% endcontent-ref %}

## Reference

* [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
