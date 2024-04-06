# FZ - NFC

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova** [**hacktricks repou**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repou**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Uvod <a href="#id-9wrzi" id="id-9wrzi"></a>

Za informacije o RFID i NFC proverite sledeću stranicu:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## Podržane NFC kartice <a href="#id-9wrzi" id="id-9wrzi"></a>

{% hint style="danger" %}
Osim NFC kartica, Flipper Zero podržava **druge vrste kartica visoke frekvencije** kao što su nekoliko **Mifare** Classic i Ultralight i **NTAG**.
{% endhint %}

Novi tipovi NFC kartica će biti dodati na listu podržanih kartica. Flipper Zero podržava sledeće **NFC kartice tipa A** (ISO 14443A):

* ﻿**Bankarske kartice (EMV)** — samo čita UID, SAK i ATQA bez čuvanja.
* ﻿**Nepoznate kartice** — čita (UID, SAK, ATQA) i emulira UID.

Za **NFC kartice tipa B, tipa F i tipa V**, Flipper Zero može pročitati UID bez čuvanja.

### NFC kartice tipa A <a href="#uvusf" id="uvusf"></a>

#### Bankarska kartica (EMV) <a href="#kzmrp" id="kzmrp"></a>

Flipper Zero može samo pročitati UID, SAK, ATQA i sačuvane podatke na bankarskim karticama **bez čuvanja**.

Ekran čitanja bankarske karticeZa bankarske kartice, Flipper Zero može samo pročitati podatke **bez čuvanja i emuliranja**.

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-26-31.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=916&#x26;w=2662" alt=""><figcaption></figcaption></figure>

#### Nepoznate kartice <a href="#id-37eo8" id="id-37eo8"></a>

Kada Flipper Zero **nije u mogućnosti da odredi tip NFC kartice**, tada se može pročitati i sačuvati samo **UID, SAK i ATQA**.

Ekran čitanja nepoznate karticeZa nepoznate NFC kartice, Flipper Zero može emulirati samo UID.

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-27-53.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=932&#x26;w=2634" alt=""><figcaption></figcaption></figure>

### NFC kartice tipova B, F i V <a href="#wyg51" id="wyg51"></a>

Za **NFC kartice tipova B, F i V**, Flipper Zero može samo **pročitati i prikazati UID** bez čuvanja.

<figure><img src="https://archbee.imgix.net/3StCFqarJkJQZV-7N79yY/zBU55Fyj50TFO4U7S-OXH_screenshot-2022-08-12-at-182540.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=1080&#x26;w=2704" alt=""><figcaption></figcaption></figure>

## Akcije

Za uvod o NFC-u [**pročitajte ovu stranicu**](../pentesting-rfid.md#high-frequency-rfid-tags-13.56-mhz).

### Čitanje

Flipper Zero može **čitati NFC kartice**, međutim, **ne razume sve protokole** koji se zasnivaju na ISO 14443. Međutim, pošto je **UID atribut niskog nivoa**, možete se naći u situaciji kada je **UID već pročitan, ali protokol visokog nivoa prenosa podataka je još uvek nepoznat**. Možete čitati, emulirati i ručno uneti UID koristeći Flipper za primitivne čitače koji koriste UID za autorizaciju.

#### Čitanje UID-a NASPRAM Čitanja Podataka Unutar <a href="#reading-the-uid-vs-reading-the-data-inside" id="reading-the-uid-vs-reading-the-data-inside"></a>

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

U Flipperu, čitanje tagova na 13.56 MHz može se podeliti na dva dela:

* **Čitanje niskog nivoa** — čita samo UID, SAK i ATQA. Flipper pokušava da pretpostavi protokol visokog nivoa na osnovu ovih podataka pročitanih sa kartice. Ne možete biti 100% sigurni u ovo, jer je to samo pretpostavka zasnovana na određenim faktorima.
* **Čitanje visokog nivoa** — čita podatke iz memorije kartice koristeći određeni protokol visokog nivoa. To bi bilo čitanje podataka na Mifare Ultralight, čitanje sektora sa Mifare Classic ili čitanje atributa kartice iz PayPass/Apple Pay.

### Čitanje Specifično

U slučaju da Flipper Zero nije sposoban da pronađe tip kartice iz podataka niskog nivoa, u `Dodatnim akcijama` možete odabrati `Čitanje Specifičnog Tipa Kartice` i **ručno** **označiti tip kartice koji želite pročitati**.

#### EMV Bankarske Kartice (PayPass, payWave, Apple Pay, Google Pay) <a href="#emv-bank-cards-paypass-paywave-apple-pay-google-pay" id="emv-bank-cards-paypass-paywave-apple-pay-google-pay"></a>

Osim što jednostavno čita UID, možete izvući mnogo više podataka sa bankarske kartice. Moguće je **dobiti puni broj kartice** (16 cifara na prednjoj strani kartice), **datum važenja**, a u nekim slučajevima čak i **ime vlasnika** zajedno sa listom **najnovijih transakcija**.\
Međutim, **ne možete pročitati CVV na ovaj način** (3 cifre na poleđini kartice). Takođe, **bankarske kartice su zaštićene od napada ponovnog reprodukovanja**, tako da kopiranje sa Flipperom i zatim pokušaj emuliranja za plaćanje neće raditi.

## Reference

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite pristupiti **najnovijoj verziji PEASS-a ili preuzeti HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili **telegram grupi** ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova** [**hacktricks repozitorijumu**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijumu**](https://github.com/carlospolop/hacktricks-cloud).

</details>
