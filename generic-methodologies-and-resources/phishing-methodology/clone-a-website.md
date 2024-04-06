# Clone a Website

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Za procenu phishing-a ponekad može biti korisno potpuno **klonirati veb sajt**.

Imajte na umu da možete dodati i neke payload-e na klonirani veb sajt, kao što je BeEF kuka za "kontrolu" taba korisnika.

Postoje različiti alati koje možete koristiti u tu svrhu:

### wget

```
wget -mk -nH
```

### goclone

goclone je alatka koja omogućava kloniranje web stranica. Ova tehnika se često koristi u phishing napadima kako bi se stvorila lažna kopija legitimne web stranice i prevarila korisnike da otkriju svoje poverljive informacije.

Da biste koristili goclone, prvo morate instalirati alatku na svoj sistem. Možete je preuzeti sa zvaničnog repozitorijuma goclone-a na GitHub-u.

Nakon instalacije, možete pokrenuti goclone komandom `goclone` u terminalu. Ova komanda će otvoriti interaktivni interfejs koji vam omogućava da unesete URL ciljane web stranice koju želite da klonirate.

Kada unesete URL, goclone će preuzeti sve resurse sa ciljane web stranice, uključujući HTML, CSS, JavaScript i slike. Zatim će generisati lokalnu kopiju web stranice na vašem sistemu.

Važno je napomenuti da je kloniranje web stranica ilegalno bez pristanka vlasnika web stranice. Ova tehnika se može koristiti samo u legitimne svrhe, kao što je testiranje sigurnosti ili obuka osoblja za prepoznavanje phishing napada.

Kada koristite goclone ili bilo koju drugu alatku za kloniranje web stranica, uvek budite pažljivi i poštujte zakone i etičke smernice.

```bash
#https://github.com/imthaghost/goclone
goclone <url>
```

### Alat za socijalno inženjerstvo

#### Kloniranje veb sajta

Kloniranje veb sajta je tehnika socijalnog inženjeringa koja se koristi za prevaru korisnika tako što se napravi lažna kopija legitimnog veb sajta. Ova tehnika se često koristi u phishing napadima kako bi se prikupili osetljivi podaci kao što su korisnička imena, lozinke i finansijski podaci.

Da biste klonirali veb sajt, možete koristiti alate kao što su `httrack` ili `wget`. Ovi alati omogućavaju preuzimanje celog veb sajta, uključujući HTML, CSS, JavaScript i druge resurse.

Kada preuzmete veb sajt, možete ga hostovati na svom serveru ili na cloud platformi kao što je AWS ili GCP. Zatim možete promeniti neke delove veb sajta kako biste prevarili korisnike da unesu svoje osetljive podatke.

Važno je napomenuti da je kloniranje veb sajta ilegalno i može imati ozbiljne pravne posledice. Ova tehnika se sme koristiti samo u okviru legalnih aktivnosti kao što je testiranje bezbednosti ili obuka osoblja o prepoznavanju phishing napada.

```bash
#https://github.com/trustedsec/social-engineer-toolkit
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
