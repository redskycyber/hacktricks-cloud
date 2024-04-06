# FZ - Infrared

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Uvod <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

Za više informacija o tome kako funkcioniše infracrveno svetlo, pogledajte:

{% content-ref url="../infrared.md" %}
[infrared.md](../infrared.md)
{% endcontent-ref %}

## IR Signal Receiver u Flipper Zero <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

Flipper koristi digitalni IR signalni prijemnik TSOP, koji **omogućava presretanje signala sa IR daljinskih upravljača**. Postoje neki **pametni telefoni** poput Xiaomija, koji takođe imaju IR port, ali imajte na umu da **većina njih može samo da šalje** signale i **ne može da ih prima**.

Flipperov infracrveni prijemnik je prilično osetljiv. Možete čak **uhvatiti signal** dok se nalazite **negde između** daljinskog upravljača i televizora. Nije potrebno usmeravati daljinski upravljač direktno prema IR portu Flippera. Ovo je korisno kada neko menja kanale dok stoji blizu televizora, a i vi i Flipper se nalazite na nekoj udaljenosti.

Pošto se **dekodiranje infracrvenog** signala dešava na **softverskoj** strani, Flipper Zero potencijalno podržava **prijem i slanje bilo kojih IR kodova daljinskih upravljača**. U slučaju **nepoznatih** protokola koji se ne mogu prepoznati - on **snima i reprodukuje** sirovi signal tačno onako kako je primljen.

## Akcije

### Univerzalni daljinski upravljači

Flipper Zero se može koristiti kao **univerzalni daljinski upravljač za kontrolu bilo kog televizora, klima uređaja ili media centra**. U ovom režimu, Flipper **bruteforcuje** sve **poznate kodove** svih podržanih proizvođača **prema rečniku sa SD kartice**. Nije vam potrebno odabrati određeni daljinski upravljač da biste isključili televizor u restoranu.

Dovoljno je pritisnuti dugme za napajanje u režimu Univerzalnog daljinskog upravljača, i Flipper će **sekvencijalno slati "Isključi"** komande svim televizorima koje poznaje: Sony, Samsung, Panasonic... i tako dalje. Kada televizor primi njegov signal, reagovaće i isključiti se.

Takav brute-force zahteva vreme. Što je rečnik veći, to će duže trajati da se završi. Nemoguće je saznati koji signal tačno je televizor prepoznao jer nema povratne informacije od televizora.

### Naučite novi daljinski upravljač

Moguće je **uhvatiti infracrveni signal** sa Flipper Zero. Ako **pronađe signal u bazi podataka**, Flipper će automatski **znati koji uređaj je u pitanju** i omogućiće vam da s njim komunicirate.\
Ako ne pronađe, Flipper može **sačuvati** signal i omogućiti vam da ga **ponovo reprodukujete**.

## Reference

* [https://blog.flipperzero.one/infrared/](https://blog.flipperzero.one/infrared/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
