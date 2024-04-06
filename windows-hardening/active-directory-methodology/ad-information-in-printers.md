# AD information in printers

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Postoji nekoliko blogova na internetu koji **ističu opasnosti ostavljanja štampača konfigurisanih sa LDAP-om sa podrazumevanim/slabim** prijavljivanjem.\
To je zato što napadač može **prevariti štampač da se autentifikuje protiv lažnog LDAP servera** (obično je dovoljan `nc -vv -l -p 444`) i da uhvati štampač **poverljive informacije o korisnicima u čistom tekstu**.

Takođe, neki štampači će sadržati **logove sa korisničkim imenima** ili čak moći **preuzeti sva korisnička imena** sa kontrolera domena.

Sve ove **poverljive informacije** i uobičajeni **nedostatak bezbednosti** čine štampače veoma interesantnim za napadače.

Neki blogovi o ovoj temi:

* [https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/](https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/)
* [https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856](https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856)

### Konfiguracija štampača

* **Lokacija**: Lista LDAP servera se nalazi na: `Mreža > Postavke LDAP > Postavljanje LDAP-a`.
* **Ponašanje**: Interfejs omogućava izmenu LDAP servera bez ponovnog unošenja akreditiva, ciljajući na korisničku udobnost ali postavljajući rizike po bezbednost.
* **Eksploatacija**: Eksploatacija uključuje preusmeravanje adrese LDAP servera na kontrolisani uređaj i iskorišćavanje funkcije "Test Connection" za hvatanje akreditiva.

### Hvatanje akreditiva

**Za detaljnije korake, pogledajte originalni** [**izvor**](https://grimhacker.com/2018/03/09/just-a-printer/)**.**

#### Metoda 1: Netcat Listener

Jednostavan netcat listener može biti dovoljan:

```bash
sudo nc -k -v -l -p 386
```

#### Metoda 2: Potpuni LDAP server sa Slapd-om

Pouzdaniji pristup uključuje postavljanje potpunog LDAP servera jer štampač izvršava null bind zatim upit pre nego što pokuša vezivanje za akreditive.

1. **Postavljanje LDAP servera**: Vodič prati korake sa [ovog izvora](https://www.server-world.info/en/note?os=Fedora\_26\&p=openldap).
2. **Ključni koraci**:

* Instalirajte OpenLDAP.
* Konfigurišite administratorsku lozinku.
* Uvezite osnovne šeme.
* Postavite ime domena na LDAP DB.
* Konfigurišite LDAP TLS.

3. **Izvršavanje LDAP servisa**: Kada je postavljen, LDAP servis se može pokrenuti korišćenjem:

```bash
slapd -d 2
```

### Reference

* [https://grimhacker.com/2018/03/09/just-a-printer/](https://grimhacker.com/2018/03/09/just-a-printer/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
