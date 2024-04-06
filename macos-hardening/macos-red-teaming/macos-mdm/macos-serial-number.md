# macOS Serial Number

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovne informacije

Apple uređaji nakon 2010. godine imaju serijske brojeve koji se sastoje od **12 alfanumeričkih karaktera**, pri čemu svaki segment prenosi određene informacije:

* **Prva 3 karaktera**: Označavaju **lokaciju proizvodnje**.
* **Karakteri 4 i 5**: Označavaju **godinu i nedelju proizvodnje**.
* **Karakteri 6 do 8**: Služe kao **jedinstveni identifikator** za svaki uređaj.
* **Poslednja 4 karaktera**: Specificiraju **modelni broj**.

Na primer, serijski broj **C02L13ECF8J2** prati ovu strukturu.

### **Lokacije proizvodnje (Prva 3 karaktera)**

Određeni kodovi predstavljaju specifične fabrike:

* **FC, F, XA/XB/QP/G8**: Različite lokacije u SAD-u.
* **RN**: Meksiko.
* **CK**: Cork, Irska.
* **VM**: Foxconn, Češka Republika.
* **SG/E**: Singapur.
* **MB**: Malezija.
* **PT/CY**: Koreja.
* **EE/QT/UV**: Tajvan.
* **FK/F1/F2, W8, DL/DM, DN, YM/7J, 1C/4H/WQ/F7**: Različite lokacije u Kini.
* **C0, C3, C7**: Specifični gradovi u Kini.
* **RM**: Obnovljeni uređaji.

### **Godina proizvodnje (4. karakter)**

Ovaj karakter varira od 'C' (predstavlja prvu polovinu 2010. godine) do 'Z' (druga polovina 2019. godine), pri čemu različita slova označavaju različite polugodišnje periode.

### **Nedelja proizvodnje (5. karakter)**

Brojevi 1-9 odgovaraju nedeljama 1-9. Slova C-Y (isključujući samoglasnike i 'S') predstavljaju nedelje 10-27. Za drugu polovinu godine, ovom broju se dodaje 26.

### **Jedinstveni identifikator (Karakteri 6 do 8)**

Ova tri broja osiguravaju da svaki uređaj, čak i istog modela i serije, ima jedinstveni serijski broj.

### **Modelni broj (Poslednja 4 karaktera)**

Ovi brojevi identifikuju specifični model uređaja.

### Reference

* [https://beetstech.com/blog/decode-meaning-behind-apple-serial-number](https://beetstech.com/blog/decode-meaning-behind-apple-serial-number)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
