# macOS System Extensions

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Sistemski ekstenzije / Endpoint Security okvir

Za razliku od Kernel ekstenzija, **sistemski ekstenzije se izvršavaju u korisničkom prostoru** umesto u kernel prostoru, smanjujući rizik od pada sistema zbog neispravnosti ekstenzija.

<figure><img src="../../../.gitbook/assets/image (1) (3) (1) (1).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Postoje tri vrste sistemskih ekstenzija: **DriverKit** ekstenzije, **Network** ekstenzije i **Endpoint Security** ekstenzije.

### **DriverKit ekstenzije**

DriverKit je zamena za kernel ekstenzije koje **pružaju podršku za hardver**. Omogućava da drajveri uređaja (poput USB, serijskih, NIC i HID drajvera) rade u korisničkom prostoru umesto u kernel prostoru. DriverKit okvir uključuje **verzije određenih I/O Kit klasa u korisničkom prostoru**, a kernel prosleđuje normalne I/O Kit događaje u korisnički prostor, pružajući sigurnije okruženje za rad ovih drajvera.

### **Network ekstenzije**

Network ekstenzije omogućavaju prilagođavanje mrežnih ponašanja. Postoji nekoliko vrsta Network ekstenzija:

* **App Proxy**: Koristi se za kreiranje VPN klijenta koji implementira protokol VPN-a usmeren na tokove (flows) umesto na pojedinačne pakete.
* **Packet Tunnel**: Koristi se za kreiranje VPN klijenta koji implementira protokol VPN-a usmeren na pakete umesto na tokove.
* **Filter Data**: Koristi se za filtriranje mrežnih "tokova". Može pratiti ili menjati mrežne podatke na nivou toka.
* **Filter Packet**: Koristi se za filtriranje pojedinačnih mrežnih paketa. Može pratiti ili menjati mrežne podatke na nivou paketa.
* **DNS Proxy**: Koristi se za kreiranje prilagođenog DNS provajdera. Može se koristiti za praćenje ili menjanje DNS zahteva i odgovora.

## Endpoint Security okvir

Endpoint Security je okvir koji je Apple obezbedio u macOS-u i pruža skup API-ja za sistemsku bezbednost. Namijenjen je **bezbednosnim dobavljačima i programerima za izgradnju proizvoda koji mogu pratiti i kontrolisati aktivnosti sistema** kako bi identifikovali i zaštitili se od zlonamernih aktivnosti.

Ovaj okvir pruža **kolekciju API-ja za praćenje i kontrolu aktivnosti sistema**, kao što su izvršavanje procesa, događaji sistema datotečnog sistema, mreže i kernela.

Srž ovog okvira je implementirana u kernelu, kao Kernel ekstenzija (KEXT) smeštena na **`/System/Library/Extensions/EndpointSecurity.kext`**. Ova KEXT se sastoji od nekoliko ključnih komponenti:

* **EndpointSecurityDriver**: Ovo deluje kao "ulazna tačka" za kernel ekstenziju. To je glavna tačka interakcije između OS-a i Endpoint Security okvira.
* **EndpointSecurityEventManager**: Ova komponenta je odgovorna za implementaciju kernel kuka. Kernel kuke omogućavaju okviru da prati događaje sistema presretanjem sistemskih poziva.
* **EndpointSecurityClientManager**: Ovo upravlja komunikacijom sa klijentima u korisničkom prostoru, prateći koji klijenti su povezani i trebaju primati obaveštenja o događajima.
* **EndpointSecurityMessageManager**: Ovo šalje poruke i obaveštenja o događajima klijentima u korisničkom prostoru.

Događaji koje Endpoint Security okvir može pratiti su kategorisani u:

* Događaji datoteka
* Događaji procesa
* Događaji soketa
* Događaji kernela (poput učitavanja/isključivanja kernel ekstenzije ili otvaranja I/O Kit uređaja)

### Arhitektura Endpoint Security okvira

<figure><img src="../../../.gitbook/assets/image (3) (8).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**Komunikacija u korisničkom prostoru** sa Endpoint Security okvirom se odvija putem klase IOUserClient. Koriste se dve različite podklase, u zavisnosti od vrste pozivaoca:

* **EndpointSecurityDriverClient**: Ovo zahteva `com.apple.private.endpoint-security.manager` dozvolu, koju poseduje samo sistemski proces `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Ovo zahteva `com.apple.developer.endpoint-security.client` dozvolu. Ovo bi obično koristio sigurnosni softver treće strane koji treba da komunicira sa Endpoint Security okvirom.

Endpoint Security ekstenzije:**`libEndpointSecurity.dylib`** je C biblioteka koju sistemski ekstenzije koriste za komunikaciju sa kernelom. Ova biblioteka koristi I/O Kit (`IOKit`) za komunikaciju sa Endpoint Security KEXT-om.

**`endpointsecurityd`** je ključni sistemski daemon koji je uključen u upravljanje i pokretanje sistemskih ekstenzija za bezbednost, posebno tokom ranog procesa pokretanja. Samo sistemski ekstenzije označene sa **`NSEndpointSecurityEarlyBoot`** u svom `Info.plist` fajlu dobijaju ovaj tretman ranog pokretanja.

Još jedan sistemski daemon, **`sysextd`**, **validira sistemskie ekstenzije** i premješta ih na odgovarajuća sistemsko mesto. Zatim traži odgovarajućem daemonu da učita ekstenziju. **`SystemExtensions.framework`** je odgovoran za aktiviranje i deaktiviranje sistemskih ekstenzija.

## Zaobilaženje ESF

ESF se koristi od strane sigurnosnih alata koji će pokušati da otkriju red timera, pa je svaka informacija o tome kako se to može izbeći zanimljiva.

### CVE-2021-30965

Stvar je u tome da sigurnosna aplikacija mora imati **dozvole za pun pristup disku**. Dakle, ako napadač može to ukloniti, može sprečiti pokretanje softvera:

```bash
tccutil reset All
```

Za **više informacija** o ovom zaobilazenju i srodnim, pogledajte predavanje [#OBTS v5.0: "Achillesova peta EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

Na kraju je ovo popravljeno davanjem nove dozvole **`kTCCServiceEndpointSecurityClient`** aplikaciji za sigurnost koju upravlja **`tccd`**, tako da `tccutil` neće brisati njene dozvole i sprečavaće je da se pokrene.

## Reference

* [**OBTS v3.0: "Endpoint Security & Insecurity" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
