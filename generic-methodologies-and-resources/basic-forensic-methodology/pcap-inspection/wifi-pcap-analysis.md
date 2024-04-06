# Wifi Pcap Analysis

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Provera BSSID-ova

Kada dobijete snimak čiji je glavni saobraćaj Wifi, koristeći WireShark, možete početi istraživanje svih SSID-ova snimka sa _Wireless --> WLAN Traffic_:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

### Brute Force

Jedna od kolona na tom ekranu ukazuje da li je **pronađena bilo kakva autentifikacija unutar pcap-a**. Ako je to slučaj, možete pokušati Brute Force koristeći `aircrack-ng`:

```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```

Na primer, povratit će WPA lozinku koja štiti PSK (pre-shared key), koja će biti potrebna kasnije za dešifrovanje saobraćaja.

## Podaci u balisama / bočni kanal

Ako sumnjate da **podaci cure unutar balisa WiFi mreže**, možete proveriti balise mreže koristeći filter poput sledećeg: `wlan contains <IMEmreže>`, ili `wlan.ssid == "IMEmreže"` pretražite filtrirane pakete u potrazi za sumnjivim nizovima.

## Pronalaženje nepoznatih MAC adresa u WiFi mreži

Sledeći link će biti koristan za pronalaženje **mašina koje šalju podatke unutar WiFi mreže**:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

Ako već znate **MAC adrese, možete ih ukloniti iz rezultata** dodavanjem provere poput ove: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Kada ste otkrili **nepoznate MAC adrese** koje komuniciraju unutar mreže, možete koristiti **filtere** poput sledećeg: `wlan.addr==<MAC adresa> && (ftp || http || ssh || telnet)` da biste filtrirali njen saobraćaj. Imajte na umu da su filteri ftp/http/ssh/telnet korisni ako ste dešifrovali saobraćaj.

## Dešifrovanje saobraćaja

Edit --> Preferences --> Protocols --> IEEE 802.11--> Edit

![](<../../../.gitbook/assets/image (426).png>)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite videti **oglašavanje vaše kompanije u HackTricks-u** ili **preuzeti HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
