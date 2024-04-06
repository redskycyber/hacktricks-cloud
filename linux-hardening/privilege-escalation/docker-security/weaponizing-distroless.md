# Weaponizing Distroless

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Šta je Distroless

Distroless kontejner je vrsta kontejnera koji **sadrži samo neophodne zavisnosti za pokretanje određene aplikacije**, bez dodatnog softvera ili alata koji nisu potrebni. Ovi kontejneri su dizajnirani da budu što **lakši** i **sigurniji** mogući, i cilj im je da **minimiziraju površinu napada** uklanjanjem svih nepotrebnih komponenti.

Distroless kontejneri se često koriste u **proizvodnim okruženjima gde je sigurnost i pouzdanost od velike važnosti**.

Neki **primeri** distroless kontejnera su:

* Pruženi od strane **Google-a**: [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* Pruženi od strane **Chainguard-a**: [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## Oružavanje Distroless

Cilj oružavanja distroless kontejnera je da se omogući **izvršavanje proizvoljnih binarnih fajlova i payload-ova čak i sa ograničenjima** koja nameće **distroless** (nedostatak uobičajenih binarnih fajlova u sistemu) i takođe zaštite koje se često nalaze u kontejnerima kao što su **samo za čitanje** ili **bez izvršavanja** u `/dev/shm`.

### Kroz memoriju

Dolazi u nekom trenutku 2023...

### Putem postojećih binarnih fajlova

#### openssl

\*\*\*\*[**U ovom postu,**](https://www.form3.tech/engineering/content/exploiting-distroless-images) objašnjeno je da se binarni fajl **`openssl`** često nalazi u ovim kontejnerima, potencijalno zato što je **potreban** softveru koji će se pokretati unutar kontejnera.

Zloupotrebom binarnog fajla **`openssl`** moguće je **izvršiti proizvoljne stvari**.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
