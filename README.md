# HackTricks Wolke

<details>

<summary><strong>Leer AWS-hacking van nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy jou **maatskappy geadverteer wil sien in HackTricks** of **HackTricks in PDF wil aflaai**, kyk na die [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**The PEASS Family**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** my op **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Deel jou hacking-truuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-opslagplekke.

</details>

<figure><img src=".gitbook/assets/cloud.gif" alt=""><figcaption></figcaption></figure>

_Hacktricks-logo's en beweging ontwerp deur_ [_@ppiernacho_](https://www.instagram.com/ppieranacho/)_._

{% hint style="success" %}
Welkom by die bladsy waar jy elke **hacking-truuk/tegniek/watookalverwant aan CI/CD & Cloud** sal vind wat ek geleer het in **CTFs**, **werklike** lewe **omgewings**, **navorsing**, en **lees** van navorsing en nuus.
{% endhint %}

## **Pentesting CI/CD Metodologie**

**In die HackTricks CI/CD Metodologie sal jy vind hoe om infrastruktuur wat verband hou met CI/CD-aktiwiteite te pentest.** Lees die volgende bladsy vir 'n **inleiding:**

{% content-ref url="pentesting-ci-cd/pentesting-ci-cd-methodology.md" %}
[pentesting-ci-cd-methodology.md](pentesting-ci-cd/pentesting-ci-cd-methodology.md)
{% endcontent-ref %}

## Pentesting Cloud Metodologie

**In die HackTricks Cloud Metodologie sal jy vind hoe om wolkomgewings te pentest.** Lees die volgende bladsy vir 'n **inleiding:**

{% content-ref url="pentesting-cloud/pentesting-cloud-methodology.md" %}
[pentesting-cloud-methodology.md](pentesting-cloud/pentesting-cloud-methodology.md)
{% endcontent-ref %}

## Lisensie

**Copyright © Carlos Polop 2023. Behalwe waar anders gespesifiseer (die eksterne inligting wat in die boek gekopieer is, behoort aan die oorspronklike outeurs), is die teks op** [**HACK TRICKS CLOUD**](https://github.com/carlospolop/hacktricks-cloud) **deur Carlos Polop gelisensieer onder die**[ **Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**](https://creativecommons.org/licenses/by-nc/4.0/)**.**\
**As jy dit met kommersiële doeleindes wil gebruik, kontak my.**

## **Vrywaring**

{% hint style="danger" %}
Hierdie boek, 'HackTricks Cloud,' is slegs bedoel vir opvoedkundige en inligtingsdoeleindes. Die inhoud van hierdie boek word verskaf op 'n 'soos dit is' basis, en die outeurs en uitgewers maak geen voorstellings of waarborge van enige aard, uitdruklik of stilswyend, oor die volledigheid, akkuraatheid, betroubaarheid, geskiktheid of beskikbaarheid van die inligting, produkte, dienste, of verwante grafika binne hierdie boek nie. Enige vertroue wat jy plaas op sulke inligting is dus streng op jou eie risiko.

Die outeurs en uitgewers sal in geen geval aanspreeklik wees vir enige verlies of skade, insluitend sonder beperking, indirekte of gevolglike verlies of skade, of enige verlies of skade wat voortspruit uit verlies van data of wins wat voortspruit uit die gebruik van hierdie boek.

Verder word die tegnieke en wenke wat in hierdie boek beskryf word, slegs verskaf vir opvoedkundige en inligtingsdoeleindes, en moet nie gebruik word vir enige onwettige of skadelike aktiwiteite nie. Die outeurs en uitgewers keur geen onwettige of onetiese aktiwiteite goed of ondersteun dit nie, en enige gebruik van die inligting wat binne hierdie boek vervat is, is op die gebruiker se eie risiko en diskresie.

Die gebruiker is uitsluitlik verantwoordelik vir enige aksies wat geneem word op grond van die inligting wat binne hierdie boek vervat is, en moet altyd professionele advies en bystand soek wanneer daar gepoog word om enige van die tegnieke of wenke wat hierin beskryf word, te implementeer.

Deur hierdie boek te gebruik, stem die gebruiker daarmee in om die outeurs en uitgewers vry te stel van enige aanspreeklikheid en verantwoordelikheid vir enige skade, verliese, of skade wat mag voortspruit uit die gebruik van hierdie boek of enige van die inligting wat daarin vervat is.
{% endhint %}

<details>

<summary><strong>Leer AWS-hacking van nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy jou **maatskappy geadverteer wil sien in HackTricks** of **HackTricks in PDF wil aflaai**, kyk na die [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**The PEASS Family**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** my op **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Deel jou hacking-truuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-opslagplekke.

</details>
