# AD CS Account Persistence

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Ovo je kratak rezime poglavlja o održivosti mašine iz fantastičnog istraživanja sa** [**https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf**](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)

## **Razumevanje krađe aktivnih korisničkih akreditacija putem sertifikata - PERSIST1**

U scenariju gde korisnik može zatražiti sertifikat koji omogućava autentifikaciju domena, napadač ima priliku da **zatraži** i **ukrade** ovaj sertifikat kako bi održao održivost na mreži. Podrazumevano, šablona `User` u Active Directory-ju omogućava takve zahteve, mada se ponekad može onemogućiti.

Korišćenjem alata nazvanog [**Certify**](https://github.com/GhostPack/Certify), može se pretraživati za validne sertifikate koji omogućavaju trajni pristup:

```bash
Certify.exe find /clientauth
```

Naglašeno je da snaga sertifikata leži u njegovoj sposobnosti da **autentifikuje kao korisnik** kojem pripada, bez obzira na promene lozinke, sve dok sertifikat ostaje **važeći**.

Sertifikati se mogu zahtevati putem grafičkog interfejsa koristeći `certmgr.msc` ili putem komandne linije sa `certreq.exe`. Sa **Certify**-jem, proces zahtevanja sertifikata je pojednostavljen na sledeći način:

```bash
Certify.exe request /ca:CA-SERVER\CA-NAME /template:TEMPLATE-NAME
```

Nakon uspešnog zahteva, generiše se sertifikat zajedno sa privatnim ključem u `.pem` formatu. Da biste to pretvorili u `.pfx` datoteku, koja se može koristiti na Windows sistemima, koristi se sledeća komanda:

```bash
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

`.pfx` datoteku možete preneti na ciljni sistem i koristiti je sa alatom nazvanim [**Rubeus**](https://github.com/GhostPack/Rubeus) kako biste zatražili Ticket Granting Ticket (TGT) za korisnika, produžavajući pristup napadača dok je sertifikat **važeći** (obično jednu godinu):

```bash
Rubeus.exe asktgt /user:harmj0y /certificate:C:\Temp\cert.pfx /password:CertPass!
```

Važno upozorenje je podeljeno o tome kako ova tehnika, kombinovana sa drugom metodom opisanom u odeljku **THEFT5**, omogućava napadaču da trajno dobije **NTLM hash** naloga bez interakcije sa Local Security Authority Subsystem Service (LSASS), i to iz neprivilegovanog konteksta, pružajući prikriveniju metodu za dugoročnu krađu akreditiva.

## **Dobijanje mašinske perzistencije pomoću sertifikata - PERSIST2**

Druga metoda uključuje upisivanje mašinskog naloga kompromitovanog sistema za sertifikat, koristeći podrazumevani `Machine` šablon koji dozvoljava takve akcije. Ako napadač stekne privilegije na sistemu, mogu koristiti **SYSTEM** nalog za zahtevanje sertifikata, pružajući oblik **perzistencije**:

```bash
Certify.exe request /ca:dc.theshire.local/theshire-DC-CA /template:Machine /machine
```

Ovaj pristup omogućava napadaču da se autentifikuje na **Kerberos** kao korisnički račun mašine i koristi **S4U2Self** da bi dobio Kerberos servisne tikete za bilo koji servis na hostu, efektivno dajući napadaču trajan pristup mašini.

## **Proširenje postojanosti kroz obnovu sertifikata - PERSIST3**

Poslednja metoda koja se razmatra uključuje iskorišćavanje **perioda važenja** i **perioda obnove** šablona sertifikata. Obnavljanjem sertifikata pre isteka, napadač može održavati autentifikaciju na Active Directory-ju bez potrebe za dodatnim upisima tiketa, što bi moglo ostaviti tragove na serveru za izdavanje sertifikata (CA).

Ovaj pristup omogućava **proširenu postojanost**, minimizirajući rizik od otkrivanja kroz manje interakcija sa serverom za izdavanje sertifikata i izbegavanje generisanja artefakata koji bi mogli upozoriti administratore na upad.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
