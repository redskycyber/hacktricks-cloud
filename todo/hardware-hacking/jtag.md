# JTAG

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## JTAGenum

[**JTAGenum** ](https://github.com/cyphunk/JTAGenum)je alat koji se može koristiti sa Raspberry PI ili Arduino uređajem kako bi se pronašli JTAG pinovi na nepoznatom čipu.\
U **Arduino**-u, povežite **pinove od 2 do 11 sa 10 pinova koji potencijalno pripadaju JTAG-u**. Učitajte program u Arduino i on će pokušati da brute force metodom proveri sve pinove kako bi pronašao da li neki od njih pripada JTAG-u i koji je svaki od njih.\
U **Raspberry PI**-ju možete koristiti samo **pinove od 1 do 6** (6 pinova, tako da će testiranje svakog potencijalnog JTAG pina biti sporije).

### Arduino

U Arduino-u, nakon što povežete kablove (pin 2 do 11 sa JTAG pinovima i Arduino GND sa baznom pločom GND), **učitajte JTAGenum program u Arduino** i u Serial Monitor-u pošaljite **`h`** (komanda za pomoć) i trebali biste videti pomoć:

![](<../../.gitbook/assets/image (643).png>)

![](<../../.gitbook/assets/image (650).png>)

Konfigurišite **"No line ending" i 115200baud**.\
Pošaljite komandu s da biste započeli skeniranje:

![](<../../.gitbook/assets/image (651) (1) (1) (1).png>)

Ako ste povezani sa JTAG-om, pronaći ćete jednu ili više **linija koje počinju sa FOUND!** koje ukazuju na pinove JTAG-a.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
