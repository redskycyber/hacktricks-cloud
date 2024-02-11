# HackTricks Cloud

<details>

<summary><strong>Jifunze kuhusu kudukua AWS kutoka sifuri hadi shujaa na</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Mtaalam wa Timu Nyekundu ya AWS ya HackTricks)</strong></a><strong>!</strong></summary>

Njia nyingine za kusaidia HackTricks:

* Ikiwa unataka kuona **kampuni yako ikitangazwa kwenye HackTricks** au **kupakua HackTricks kwa PDF** Angalia [**MIPANGO YA KUJIUNGA**](https://github.com/sponsors/carlospolop)!
* Pata [**bidhaa rasmi za PEASS & HackTricks**](https://peass.creator-spring.com)
* Gundua [**Familia ya PEASS**](https://opensea.io/collection/the-peass-family), mkusanyiko wetu wa kipekee wa [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Jiunge na** 💬 [**Kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au kikundi cha [**telegram**](https://t.me/peass) au **nifuata** kwenye **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Shiriki mbinu zako za kudukua kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>

<figure><img src=".gitbook/assets/cloud.gif" alt=""><figcaption></figcaption></figure>

_Logo za Hacktricks & motion zilizoandaliwa na_ [_@ppiernacho_](https://www.instagram.com/ppieranacho/)_._

{% hint style="success" %}
Karibu kwenye ukurasa ambapo utapata kila **mbinu/tekinolojia/chochote kinachohusiana na CI/CD & Cloud** niliyojifunza katika **CTFs**, **mazingira ya** halisi, **utafiti**, na **kusoma** utafiti na habari.
{% endhint %}

## **Mbinu ya Kudukua CI/CD**

**Katika Mbinu ya Kudukua CI/CD ya HackTricks utapata jinsi ya kudukua miundombinu inayohusiana na shughuli za CI/CD.** Soma ukurasa ufuatao kwa **utangulizi:**

{% content-ref url="pentesting-ci-cd/pentesting-ci-cd-methodology.md" %}
[pentesting-ci-cd-methodology.md](pentesting-ci-cd/pentesting-ci-cd-methodology.md)
{% endcontent-ref %}

## Mbinu ya Kudukua Cloud

**Katika Mbinu ya Kudukua Cloud ya HackTricks utapata jinsi ya kudukua mazingira ya wingu.** Soma ukurasa ufuatao kwa **utangulizi:**

{% content-ref url="pentesting-cloud/pentesting-cloud-methodology.md" %}
[pentesting-cloud-methodology.md](pentesting-cloud/pentesting-cloud-methodology.md)
{% endcontent-ref %}

## Leseni

**Haki miliki © Carlos Polop 2023. Isipokuwa pale inapobainishwa vinginevyo (habari za nje zilizochapishwa kwenye kitabu zinamilikiwa na waandishi wa asili), maandishi kwenye** [**HACK TRICKS CLOUD**](https://github.com/carlospolop/hacktricks-cloud) **na Carlos Polop inamilikiwa chini ya** [**Aina ya Kutambulisha-Kutokufanya Biashara 4.0 Kimataifa (CC BY-NC 4.0)**](https://creativecommons.org/licenses/by-nc/4.0/)**.**\
**Ikiwa unataka kutumia kwa madhumuni ya kibiashara, wasiliana nami.**

## **Taarifa ya Kisheria**

{% hint style="danger" %}
Kitabu hiki, 'HackTricks Cloud,' kinalenga kwa madhumuni ya elimu na habari pekee. Yaliyomo ndani ya kitabu hiki yanatolewa kwa msingi wa 'kama ilivyo,' na waandishi na wachapishaji hawatoi uwakilishi au dhamana ya aina yoyote, wazi au iliyopendekezwa, kuhusu ukamilifu, usahihi, uaminifu, ufaa, au upatikanaji wa habari, bidhaa, huduma, au michoro inayohusiana iliyomo ndani ya kitabu hiki. Kuiamini habari kama hiyo kunategemea kabisa hatari yako mwenyewe.

Waandishi na wachapishaji hawatawajibika kamwe kwa upotezaji au uharibifu, ikiwa ni pamoja na bila kikomo, upotezaji au uharibifu usio wa moja kwa moja au wa matokeo, au upotezaji au uharibifu wowote unaotokana na upotezaji wa data au faida unaotokana na, au kuhusiana na, matumizi ya kitabu hiki.

Zaidi ya hayo, mbinu na vidokezo vilivyoelezwa katika kitabu hiki vinatolewa kwa madhumuni ya elimu na habari pekee, na havipaswi kutumiwa kwa shughuli yoyote haramu au yenye nia mbaya. Waandishi na wachapishaji hawakubaliani au kusaidia shughuli yoyote haramu au isiyo ya maadili, na matumizi yoyote ya habari iliyomo ndani ya kitabu hiki ni kwa hatari na uamuzi wa mtumiaji mwenyewe.

Mtumiaji ndiye anayewajibika peke yake kwa hatua zozote zinazochukuliwa kulingana na habari iliyomo ndani ya kitabu hiki, na daima anapaswa kutafuta ushauri wa kitaalam na msaada anapojaribu kutekeleza mbinu au vidokezo vilivyoelezwa hapa.

Kwa kutumia kitabu hiki, mtumiaji anakubaliana kuachilia waandishi na wachapishaji kutoka kwa dhima na jukumu lolote kwa uharibifu, hasara, au madhara yanayoweza kutokea kutokana na matumizi ya kitabu hiki au habari yoyote iliyomo ndani yake.
{% endhint %}

<details>

<summary><strong>Jifunze kuhusu kudukua AWS kutoka sifuri hadi shujaa na</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Mtaalam wa Timu Nyekundu ya AWS ya HackTricks)</strong></a><strong>!</strong></summary>

Njia nyingine za kusaidia HackTricks:

* Ikiwa unataka kuona **kampuni yako ikitangazwa kwenye HackTricks** au **kupakua HackTricks kwa PDF** Angalia [**MIPANGO YA KUJIUNGA**](https://github.com/sponsors/carlospolop)!
* Pata [**bidhaa rasmi za PEASS & HackTricks**](https://peass.creator-spring.com)
* Gundua [**Familia ya PEASS**](https://opensea.io/collection/the-peass-family), mkusanyiko wetu wa kipekee wa [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Jiunge na** 💬 [**Kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au kikundi cha [**telegram**](https://t.me/peass) au **nifuata** kwenye **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Shiriki mbinu zako za kudukua kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
