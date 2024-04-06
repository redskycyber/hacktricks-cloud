# macOS Kernel & System Extensions

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## XNU Kernel

**Srce macOS-a je XNU**, što označava "X is Not Unix". Ovaj kernel je suštinski sastavljen od **Mach mikrokernela** (o kojem će biti reči kasnije), **i** elemenata iz Berkeley Software Distribution (**BSD**). XNU takođe pruža platformu za **kernel drivere putem sistema nazvanog I/O Kit**. XNU kernel je deo Darwin open source projekta, što znači da je **njegov izvorni kod slobodno dostupan**.

Iz perspektive istraživača bezbednosti ili Unix programera, **macOS** može delovati prilično **slično** sistemu **FreeBSD** sa elegantnim grafičkim korisničkim interfejsom i nizom prilagođenih aplikacija. Većina aplikacija razvijenih za BSD će se kompajlirati i pokrenuti na macOS-u bez potrebe za modifikacijama, jer su alati komandne linije poznati Unix korisnicima prisutni u macOS-u. Međutim, zbog toga što XNU kernel inkorporira Mach, postoje značajne razlike između tradicionalnog Unix-sličnog sistema i macOS-a, a ove razlike mogu izazvati potencijalne probleme ili pružiti jedinstvene prednosti.

Izvorni kod XNU-a: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach je **mikrokernel** dizajniran da bude **kompatibilan sa UNIX-om**. Jedan od njegovih ključnih dizajnerskih principa bio je da **minimizira** količinu **koda** koji se izvršava u **kernel** prostoru i umesto toga omogući mnogim tipičnim funkcijama kernela, poput sistema datoteka, mreženja i I/O-a, da se **izvršavaju kao zadaci na nivou korisnika**.

U XNU-u, Mach je **odgovoran za mnoge kritične operacije na niskom nivou** koje tipično rukuje kernel, poput raspoređivanja procesora, multitaskinga i upravljanja virtuelnom memorijom.

### BSD

XNU **kernel** takođe **inkorporira** značajnu količinu koda koji potiče iz projekta **FreeBSD**. Ovaj kod **se izvršava kao deo kernela zajedno sa Mach-om**, u istom adresnom prostoru. Međutim, FreeBSD kod unutar XNU-a može biti značajno drugačiji od originalnog FreeBSD koda jer su bile potrebne modifikacije kako bi se osigurala njegova kompatibilnost sa Mach-om. FreeBSD doprinosi mnogim operacijama kernela uključujući:

* Upravljanje procesima
* Obrada signala
* Osnovni mehanizmi bezbednosti, uključujući upravljanje korisnicima i grupama
* Infrastruktura sistemskih poziva
* TCP/IP stek i soketi
* Firewall i filtriranje paketa

Razumevanje interakcije između BSD-a i Mach-a može biti kompleksno, zbog njihovih različitih konceptualnih okvira. Na primer, BSD koristi procese kao svoju osnovnu izvršnu jedinicu, dok Mach funkcioniše na osnovu niti. Ova razlika je usklađena u XNU-u tako što **svaki BSD proces povezuje sa Mach zadatkom** koji sadrži tačno jednu Mach nit. Kada se koristi BSD-ov sistemski poziv fork(), BSD kod unutar kernela koristi Mach funkcije za kreiranje strukture zadatka i niti.

Štaviše, **Mach i BSD održavaju različite modele bezbednosti**: **Mach-ov** model bezbednosti se zasniva na **pravima porta**, dok BSD-ov model bezbednosti funkcioniše na osnovu **vlasništva procesa**. Razlike između ova dva modela ponekad su rezultirale ranjivostima lokalnog eskaliranja privilegija. Osim tipičnih sistemskih poziva, postoje i **Mach zamke koje omogućavaju programima na nivou korisnika da interaguju sa kernelom**. Ovi različiti elementi zajedno čine složenu, hibridnu arhitekturu macOS kernela.

### I/O Kit - Drajveri

I/O Kit je open-source, objektno orijentisani **framework za drajvere uređaja** u XNU kernelu, koji rukuje **dinamički učitavanim drajverima uređaja**. Omogućava dodavanje modularnog koda u kernel u hodu, podržavajući različit hardver.

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - Međuprocesna komunikacija

{% content-ref url="../macos-proces-abuse/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../macos-proces-abuse/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Kernelcache

**Kernelcache** je **prekompajlirana i prelinkovana verzija XNU kernela**, zajedno sa esencijalnim drajverima uređaja i kernel proširenjima. Čuva se u **kompresovanom** formatu i dekompresuje u memoriju tokom procesa pokretanja. Kernelcache omogućava **brže vreme pokretanja** imajući spremnu verziju kernela i ključnih drajvera, smanjujući vreme i resurse koji bi inače bili potrošeni na dinamičko učitavanje i povezivanje ovih komponenti tokom pokretanja.

Na iOS-u se nalazi u **`/System/Library/Caches/com.apple.kernelcaches/kernelcache`**, a na macOS-u ga možete pronaći sa **`find / -name kernelcache 2>/dev/null`** ili **`mdfind kernelcache | grep kernelcache`**

Moguće je pokrenuti **`kextstat`** da proverite učitane kernel proširenja.

#### IMG4

IMG4 format datoteke je kontejnerski format koji koristi Apple u svojim iOS i macOS uređajima za sigurno **čuvanje i proveru firmware** komponenti (poput **kernelcache**). IMG4 format uključuje zaglavlje i nekoliko oznaka koje obuhvataju različite delove podataka uključujući stvarni payload (kao što je kernel ili bootloader), potpis, i skup manifestnih svojstava. Format podržava kriptografsku verifikaciju, omogućavajući uređaju da potvrdi autentičnost i integritet firmware komponente pre njene izvršne.

Obično je sastavljen od sledećih komponenti:

* **Payload (IM4P)**:
* Često kompresovan (LZFSE4, LZSS, …)
* Opciono enkriptovan
* **Manifest (IM4M)**:
* Sadrži Potpis
* Dodatni rečnik Ključ/Vrednost
* **Informacije o obnovi (IM4R)**:
* Takođe poznato kao APNonce
* Sprječava ponovno izvođenje nekih ažuriranja
* OPCIONALNO: Obično se ovo ne nalazi

Dekompresujte Kernelcache:

```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```

#### Simboli kernel keša

Ponekad Apple objavljuje **kernel keš** sa **simbolima**. Možete preuzeti neke firmvere sa simbolima prateći linkove na [https://theapplewiki.com](https://theapplewiki.com/).

### IPSW

Ovo su Apple **firmveri** koje možete preuzeti sa [**https://ipsw.me/**](https://ipsw.me/). Pored ostalih datoteka, sadržaće **kernel keš**.\
Da biste **izvukli** datoteke, jednostavno ih **raspakujte**.

Nakon izdvajanja firmvera dobićete datoteku poput: **`kernelcache.release.iphone14`**. U **IMG4** formatu, možete izvući zanimljive informacije pomoću:

* [**pyimg4**](https://github.com/m1stadev/PyIMG4)

{% code overflow="wrap" %}
```bash
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
{% endcode %}

* [**img4tool**](https://github.com/tihmstar/img4tool)

```bash
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```

Možete proveriti izvučeni kernelcache za simbole sa: **`nm -a kernelcache.release.iphone14.e | wc -l`**

Sada možemo **izvući sve ekstenzije** ili **onu koja vas zanima:**

```bash
# List all extensions
kextex -l kernelcache.release.iphone14.e
## Extract com.apple.security.sandbox
kextex -e com.apple.security.sandbox kernelcache.release.iphone14.e

# Extract all
kextex_all kernelcache.release.iphone14.e

# Check the extension for symbols
nm -a binaries/com.apple.security.sandbox | wc -l
```

## macOS Kernel ekstenzije

macOS je **izuzetno restriktivan kada je u pitanju učitavanje Kernel ekstenzija** (.kext) zbog visokih privilegija pod kojima će se kod izvršavati. Zapravo, podrazumevano je praktično nemoguće (osim ako se ne pronađe način zaobilaženja).

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### macOS Sistem ekstenzije

Umesto korišćenja Kernel ekstenzija, macOS je kreirao Sistem ekstenzije, koje pružaju API-je na nivou korisnika za interakciju sa kernelom. Na ovaj način, programeri mogu izbeći korišćenje kernel ekstenzija.

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## Reference

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><strong>Naučite hakovanje AWS-a od početnika do stručnjaka sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA ČLANSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks merch**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
