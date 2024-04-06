# macOS Defensive Apps

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Firewall-i

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): Pratiće svaku vezu koju uspostavi svaki proces. Zavisno od moda (tiho dozvoljava veze, tiho odbija veze i upozorenje) **prikazaće vam upozorenje** svaki put kada se uspostavi nova veza. Takođe ima veoma lep grafički interfejs za prikaz svih ovih informacija.
* [**LuLu**](https://objective-see.org/products/lulu.html): Objective-See firewall. Ovo je osnovni firewall koji će vas upozoriti na sumnjive veze (ima grafički interfejs, ali nije tako fancy kao kod Little Snitch-a).

## Detekcija perzistencije

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): Objective-See aplikacija koja će pretražiti nekoliko lokacija gde **malver može biti perzistentan** (to je alat koji se koristi samo jednom, nije monitoring servis).
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): Slično kao KnockKnock, prati procese koji generišu perzistenciju.

## Detekcija keyloggers-a

* [**ReiKey**](https://objective-see.org/products/reikey.html): Objective-See aplikacija za pronalaženje **keyloggers-a** koji instaliraju "event taps" tastature.

## Detekcija ransomware-a

* [**RansomWhere**](https://objective-see.org/products/ransomwhere.html): Objective-See aplikacija za detekciju akcija **enkripcije fajlova**.

## Detekcija mikrofona i web kamere

* [**OverSight**](https://objective-see.org/products/oversight.html): Objective-See aplikacija za detekciju **aplikacija koje koriste web kameru i mikrofon**.

## Detekcija procesnog ubacivanja

* [**Shield**](https://theevilbit.github.io/shield/): Aplikacija koja **detektuje različite tehnike procesnog ubacivanja**.
