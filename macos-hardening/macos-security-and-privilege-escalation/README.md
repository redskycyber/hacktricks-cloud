# macOS Security & Privilege Escalation

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite nam se na** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Pridružite se [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) serveru kako biste komunicirali sa iskusnim hakerima i lovcima na bagove!

**Hakerski uvidi**\
Uključite se u sadržaj koji istražuje uzbuđenje i izazove hakovanja

**Hakerske vesti u realnom vremenu**\
Budite u toku sa brzim svetom hakovanja kroz vesti i uvide u realnom vremenu

**Najnovije najave**\
Ostanite informisani o najnovijim pokretanjima nagrada za pronalaženje bagova i važnim ažuriranjima platforme

**Pridružite nam se na** [**Discord-u**](https://discord.com/invite/N3FrSbmwdy) i počnite da sarađujete sa vrhunskim hakerima danas!

## Osnovno o MacOS-u

Ako niste upoznati sa macOS-om, trebali biste početi da učite osnove macOS-a:

* Posebne macOS **datoteke i dozvole:**

{% content-ref url="macos-files-folders-and-binaries/" %}
[macos-files-folders-and-binaries](macos-files-folders-and-binaries/)
{% endcontent-ref %}

* Uobičajeni macOS **korisnici**

{% content-ref url="macos-users.md" %}
[macos-users.md](macos-users.md)
{% endcontent-ref %}

* **AppleFS**

{% content-ref url="macos-applefs.md" %}
[macos-applefs.md](macos-applefs.md)
{% endcontent-ref %}

* **Arhitektura** jezgra

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* Uobičajene macOS **mrežne usluge i protokoli**

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* **Opensource** macOS: [https://opensource.apple.com/](https://opensource.apple.com/)
* Da biste preuzeli `tar.gz`, promenite URL kao što je [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/) u [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)

### MacOS MDM

U kompanijama su **macOS** sistemi vrlo verovatno **upravljani MDM-om**. Stoga, sa perspektive napadača, važno je znati **kako to funkcioniše**:

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - Inspekcija, debagovanje i faziiranje

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## Zaštita MacOS-a

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## Napadna površina

### Dozvole za datoteke

Ako **proces koji se izvršava kao root** upisuje datoteku kojom može upravljati korisnik, korisnik to može zloupotrebiti kako bi **povećao privilegije**.\
To se može dogoditi u sledećim situacijama:

* Datoteka koja se koristi već je kreirana od strane korisnika (vlasnik je korisnik)
* Datoteka koja se koristi je upisiva od strane korisnika zbog grupe
* Datoteka koja se koristi nalazi se unutar direktorijuma koji je vlasništvo korisnika (korisnik može kreirati datoteku)
* Datoteka koja se koristi nalazi se unutar direktorijuma koji je vlasništvo root-a, ali korisnik ima pristup za upisivanje zbog grupe (korisnik može kreirati datoteku)

Mogućnost **kreiranja datoteke** koju će **koristiti root**, omogućava korisniku da **iskoristi njen sadržaj** ili čak da kreira **simboličke veze/hardlinkove** kako bi je usmerio na drugo mesto.

Za ovakve vrste ranjivosti ne zaboravite da **proverite ranjive `.pkg` instalere**:

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}

### Ekstenzija datoteke i rukovaoci URL šemom aplikacija

Čudne aplikacije registrovane preko ekstenzija datoteka mogu biti zloupotrebljene i različite aplikacije mogu biti registrovane za otvaranje određenih protokola

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## macOS TCC / SIP eskalacija privilegija

U macOS-u **aplikacije i binarni fajlovi mogu imati dozvole** za pristupanje fasciklama ili podešavanjima koja ih čine privilegovanijim od drugih.

Stoga, napadač koji želi uspešno kompromitovati macOS mašinu će morati **povećati svoje TCC privilegije** (ili čak **zaobići SIP**, u zavisnosti od svojih potreba).

Ove privilegije obično se dodeljuju u obliku **ovlašćenja** sa kojima je aplikacija potpisana, ili aplikacija može zatražiti neke pristupe i nakon što ih **korisnik odobri**, mogu se pronaći u **TCC bazama podataka**. Drugi način na koji proces može dobiti ove privilegije je da bude **potomak procesa** sa tim **privilegijama**, jer se obično **nasleđuju**.

Pratite ove linkove da biste pronašli različite načine za [**povećanje privilegija u TCC-u**](macos-security-protections/macos-tcc/#tcc-privesc-and-bypasses), za [**zaobilaženje TCC-a**](macos-security-protections/macos-tcc/macos-tcc-bypasses/) i kako je u prošlosti [**SIP zaobiđen**](macos-security-protections/macos-sip.md#sip-bypasses).

## macOS Tradicionalna eskalacija privilegija

Naravno, sa perspektive timova za crveno testiranje, trebali biste biti zainteresovani i za povećanje privilegija do root-a. Proverite sledeći post za neke smernice:

{% content-ref url="macos-privilege-escalation.md" %}
[macos-privilege-escalation.md](macos-privilege-escalation.md)
{% endcontent-ref %}

\## Reference

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Pridružite se [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) serveru kako biste komunicirali sa iskusnim hakerima i lovcima na bagove!

**Hacking Insights**\
Uključite se u sadržaj koji istražuje uzbuđenje i izazove hakovanja

**Real-Time Hack News**\
Budite u toku sa brzim svetom hakovanja kroz vesti i uvide u realnom vremenu

**Najnovije obaveštenja**\
Budite informisani o najnovijim pokretanjima nagrada za pronalaženje bagova i važnim ažuriranjima platforme

**Pridružite nam se na** [**Discord-u**](https://discord.com/invite/N3FrSbmwdy) i počnite da sarađujete sa vrhunskim hakerima danas!

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje trikove hakovanja slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
