# Other Web Tricks

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### Host zaglavlje

Često se serverska strana oslanja na **Host zaglavlje** da bi izvršila neke akcije. Na primer, može koristiti njegovu vrednost kao **domen za slanje zahteva za resetovanje lozinke**. Kada primite e-mail sa linkom za resetovanje lozinke, domen koji se koristi je onaj koji ste naveli u Host zaglavlju. Zatim, možete zahtevati resetovanje lozinke drugih korisnika i promeniti domen u onaj koji kontrolišete kako biste ukrali njihove kodove za resetovanje lozinke. [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

{% hint style="warning" %}
Imajte na umu da možda čak i ne morate čekati da korisnik klikne na link za resetovanje lozinke da biste dobili token, jer možda čak i **spam filteri ili drugi posrednički uređaji/botovi će kliknuti na njega da bi ga analizirali**.
{% endhint %}

### Booleans sesije

Ponekad, kada uspešno prođete neku verifikaciju, serverska strana će **samo dodati boolean vrednost "True" atributu bezbednosti vaše sesije**. Zatim, drugi endpoint će znati da li ste uspešno prošli tu proveru.\
Međutim, ako **prođete proveru** i vaša sesija dobije "True" vrednost u atributu bezbednosti, možete pokušati da **pristupite drugim resursima** koji **zavise od istog atributa**, ali za koje **ne biste trebali imati dozvole** za pristup. [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### Funkcionalnost registracije

Pokušajte da se registrujete kao već postojeći korisnik. Pokušajte takođe da koristite ekvivalentne karaktere (tačke, mnogo razmaka i Unicode).

### Preuzimanje e-mailova

Registrujte e-mail, pre nego što ga potvrdite promenite e-mail, zatim, ako se nova potvrda e-maila šalje na prvi registrovani e-mail, možete preuzeti bilo koji e-mail. Ili ako možete omogućiti drugi e-mail potvrđujući prvi, takođe možete preuzeti bilo koji nalog.

### Pristup internom servisnom stolu kompanija koje koriste Atlassian

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### TRACE metoda

Razvojni programeri mogu zaboraviti da onemoguće razne opcije za debagovanje u produkcionom okruženju. Na primer, HTTP `TRACE` metoda je dizajnirana u dijagnostičke svrhe. Ako je omogućena, veb server će odgovoriti na zahteve koji koriste `TRACE` metodu tako što će u odgovoru prikazati tačan zahtev koji je primljen. Ovo ponašanje često nije opasno, ali ponekad može dovesti do otkrivanja informacija, kao što je ime internih zaglavlja za autentifikaciju koja mogu biti dodata zahtevima od strane obrnutih proxy-ja.![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
