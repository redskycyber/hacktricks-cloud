# macOS SIP

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## **Osnovne informacije**

**System Integrity Protection (SIP)** u macOS-u je mehanizam dizajniran da spreči čak i najprivilegovanije korisnike da vrše neovlaštene promene u ključnim sistemskim fasciklama. Ova funkcija igra ključnu ulogu u održavanju integriteta sistema tako što ograničava radnje poput dodavanja, izmene ili brisanja fajlova u zaštićenim oblastima. Primarne fascikle koje štiti SIP uključuju:

* **/System**
* **/bin**
* **/sbin**
* **/usr**

Pravila koja definišu ponašanje SIP-a su definisana u konfiguracionom fajlu koji se nalazi na putanji **`/System/Library/Sandbox/rootless.conf`**. Unutar ovog fajla, putanje koje su prefiksirane zvezdicom (\*) označene su kao izuzeci od inače stroge SIP restrikcije.

Razmotrite sledeći primer:

```javascript
/usr
* /usr/libexec/cups
* /usr/local
* /usr/share/man
```

Ovaj odlomak implicira da iako SIP generalno obezbeđuje sigurnost direktorijuma **`/usr`**, postoje određeni poddirektorijumi (`/usr/libexec/cups`, `/usr/local` i `/usr/share/man`) gde su modifikacije dozvoljene, kako je naznačeno zvezdicom (\*) ispred njihovih putanja.

Da biste proverili da li je direktorijum ili fajl zaštićen SIP-om, možete koristiti komandu **`ls -lOd`** da biste proverili prisustvo zastavice **`restricted`** ili **`sunlnk`**. Na primer:

```bash
ls -lOd /usr/libexec/cups
drwxr-xr-x  11 root  wheel  sunlnk 352 May 13 00:29 /usr/libexec/cups
```

U ovom slučaju, zastavica **`sunlnk`** označava da se sam direktorijum `/usr/libexec/cups` **ne može izbrisati**, iako se datoteke unutar njega mogu kreirati, menjati ili brisati.

S druge strane:

```bash
ls -lOd /usr/libexec
drwxr-xr-x  338 root  wheel  restricted 10816 May 13 00:29 /usr/libexec
```

Evo, **`restricted`** oznaka ukazuje da je direktorijum `/usr/libexec` zaštićen od strane SIP-a. U zaštićenom direktorijumu, fajlovi ne mogu biti kreirani, modifikovani ili obrisani.

Osim toga, ako fajl sadrži atribut **`com.apple.rootless`** proširenog **atributa**, taj fajl će takođe biti **zaštićen od strane SIP-a**.

**SIP takođe ograničava druge root akcije** kao što su:

* Učitavanje nepouzdanih kernel ekstenzija
* Dobijanje task-portova za Apple-potpisane procese
* Modifikacija NVRAM promenljivih
* Dozvoljavanje kernel debagovanja

Opcije se čuvaju u nvram promenljivoj kao bitflag (`csr-active-config` na Intel-u i `lp-sip0` se čita iz pokrenutog Device Tree-a za ARM). Možete pronaći oznake u XNU izvornom kodu u `csr.sh`:

<figure><img src="../../../.gitbook/assets/image (720).png" alt=""><figcaption></figcaption></figure>

### SIP Status

Možete proveriti da li je SIP omogućen na vašem sistemu pomoću sledeće komande:

```bash
csrutil status
```

Ako želite da onemogućite SIP, morate ponovo pokrenuti računar u režimu oporavka (pritiskom na Command+R tokom pokretanja), a zatim izvršiti sledeću komandu:

```bash
csrutil disable
```

Ako želite da zadržite SIP omogućen, ali uklonite zaštitu od debagiranja, to možete učiniti na sledeći način:

```bash
csrutil enable --without debug
```

### Ostale restrikcije

* **Onemogućava učitavanje nepotpisanih kernel ekstenzija** (kexts), osiguravajući da samo verifikovane ekstenzije komuniciraju sa sistemskim kernelom.
* **Onemogućava debagovanje** macOS sistemskih procesa, čime se štite osnovne komponente sistema od neovlašćenog pristupa i izmena.
* **Onemogućava alate** poput dtrace da inspektuju sistemskih procesa, dodatno štiteći integritet rada sistema.

[**Saznajte više o SIP informacijama u ovom predavanju**](https://www.slideshare.net/i0n1c/syscan360-stefan-esser-os-x-el-capitan-sinking-the-ship)**.**

## Bypass-ovi SIP-a

Bypass-ovanje SIP-a omogućava napadaču da:

* **Pristupi korisničkim podacima**: Čita osetljive korisničke podatke poput mejlova, poruka i istorije Safari-ja sa svih korisničkih naloga.
* **Bypass TCC-a**: Direktno manipuliše TCC (Transparentnost, Saglasnost i Kontrola) bazom podataka kako bi dobio neovlašćen pristup veb kameri, mikrofonu i drugim resursima.
* **Ustvari postojanost**: Postavlja maliciozni softver na SIP-om zaštićene lokacije, čineći ga otpornim na uklanjanje, čak i uz privilegije root-a. Ovo takođe uključuje mogućnost manipulacije Malware Removal Tool (MRT).
* **Učitava kernel ekstenzije**: Iako postoje dodatne zaštite, bypass-ovanje SIP-a pojednostavljuje proces učitavanja nepotpisanih kernel ekstenzija.

### Installer paketi

**Installer paketi potpisani Apple-ovim sertifikatom** mogu zaobići njegove zaštite. To znači da će čak i paketi potpisani od strane standardnih programera biti blokirani ako pokušaju da izmene SIP-om zaštićene direktorijume.

### Nepostojeći SIP fajl

Jedna potencijalna rupa u sistemu je da, ako je fajl naveden u **`rootless.conf` ali trenutno ne postoji**, može biti kreiran. Malver bi mogao iskoristiti ovo da **uspostavi postojanost** na sistemu. Na primer, zlonamerni program bi mogao kreirati .plist fajl u `/System/Library/LaunchDaemons` ako je naveden u `rootless.conf` ali nije prisutan.

### com.apple.rootless.install.heritable

{% hint style="danger" %}
Odobrenje **`com.apple.rootless.install.heritable`** omogućava zaobilaženje SIP-a.
{% endhint %}

#### Shrootless

[**Istraživači sa ovog blog posta**](https://www.microsoft.com/en-us/security/blog/2021/10/28/microsoft-finds-new-macos-vulnerability-shrootless-that-could-bypass-system-integrity-protection/) otkrili su ranjivost u mehanizmu System Integrity Protection (SIP) u macOS-u, nazvanu ranjivost 'Shrootless'. Ova ranjivost se odnosi na **`system_installd`** demon, koji ima odobrenje **`com.apple.rootless.install.heritable`**, koje omogućava bilo kojem od njegovih podprocesa da zaobiđe SIP-ove restrikcije fajl sistema.

**`system_installd`** demon će instalirati pakete koji su potpisani od strane **Apple-a**.

Istraživači su otkrili da tokom instalacije Apple-ovog potpisanog paketa (.pkg fajla), **`system_installd`** **pokreće** sve **post-install** skripte koje su uključene u paket. Ove skripte se izvršavaju podrazumevanom ljuskom, **`zsh`**, koja automatski **pokreće** komande iz **`/etc/zshenv`** fajla, ako postoji, čak i u neinteraktivnom režimu. Ovo ponašanje može biti iskorišćeno od strane napadača: kreiranjem zlonamerne `/etc/zshenv` datoteke i čekanjem da **`system_installd` pozove `zsh`**, mogu se izvršiti proizvoljne operacije na uređaju.

Osim toga, otkriveno je da se \*\*`/etc/zshenv može koristiti kao opšta tehnika napada**, ne samo za zaobilaženje SIP-a. Svaki korisnički profil ima` ~~/.zshenv`datoteku, koja se ponaša na isti način kao`/etc/zshenv`, ali ne zahteva privilegije root-a. Ova datoteka može se koristiti kao mehanizam postojanosti, pokrećući se svaki put kada se` zsh`pokrene, ili kao mehanizam za podizanje privilegija. Ako admin korisnik podigne privilegije na root koristeći`sudo -s`ili`sudo `,`~~ /.zshenv\` datoteka će se pokrenuti, efektivno podižući privilegije na root.

#### [**CVE-2022-22583**](https://perception-point.io/blog/technical-analysis-cve-2022-22583/)

U [**CVE-2022-22583**](https://perception-point.io/blog/technical-analysis-cve-2022-22583/) otkriveno je da isti **`system_installd`** proces i dalje može biti zloupotrebljen jer je stavljao **post-install skriptu unutar nasumično nazvanog foldera zaštićenog SIP-om unutar `/tmp`**. Stvar je u tome što **`/tmp` sam po sebi nije zaštićen SIP-om**, pa je bilo moguće **montirati** virtuelnu sliku na njega, zatim **installer** bi tu stavio **post-install skriptu**, **demontirao** virtuelnu sliku, **ponovo kreirao** sve **foldere** i **dodao** **post-installation** skriptu sa **payload-om** za izvršavanje.

#### [fsck\_cs alatka](https://www.theregister.com/2016/03/30/apple\_os\_x\_rootless/)

Identifikovana je ranjivost gde je **`fsck_cs`** bio zaveden da ošteti ključni fajl zbog svoje sposobnosti da prati **simboličke linkove**. Konkretno, napadači su kreirali link od _`/dev/diskX`_ do fajla `/System/Library/Extensions/AppleKextExcludeList.kext/Contents/Info.plist`. Izvršavanje **`fsck_cs`** na _`/dev/diskX`_ dovelo je do oštećenja `Info.plist` fajla. Integritet ovog fajla je od vitalnog značaja za SIP (System Integrity Protection) operativnog sistema, koji kontroliše učitavanje kernel ekstenzija. Kada je oštećen, sposobnost SIP-a da upravlja isključenjima kernela je kompromitovana.

Komande za iskorišćavanje ove ranjivosti su:

```bash
ln -s /System/Library/Extensions/AppleKextExcludeList.kext/Contents/Info.plist /dev/diskX
fsck_cs /dev/diskX 1>&-
touch /Library/Extensions/
reboot
```

Eksploatacija ove ranjivosti ima ozbiljne posledice. Datoteka `Info.plist`, koja je obično odgovorna za upravljanje dozvolama za kernel ekstenzije, postaje neefikasna. To uključuje nemogućnost crnoglistinga određenih ekstenzija, poput `AppleHWAccess.kext`. Kao rezultat toga, sa mehanizmom kontrole SIP-a van funkcije, ova ekstenzija može biti učitana, omogućavajući neovlašćeni pristup čitanju i pisanju u RAM sistemu.

#### [Montiranje preko SIP zaštićenih foldera](https://www.slideshare.net/i0n1c/syscan360-stefan-esser-os-x-el-capitan-sinking-the-ship)

Bilo je moguće montirati novi fajl sistem preko **SIP zaštićenih foldera kako bi se zaobišla zaštita**.

```bash
mkdir evil
# Add contento to the folder
hdiutil create -srcfolder evil evil.dmg
hdiutil attach -mountpoint /System/Library/Snadbox/ evil.dmg
```

#### [Bypass upgrader (2016)](https://objective-see.org/blog/blog\_0x14.html)

Sistem je podešen da se pokrene sa ugrađene instalacione disk slike unutar `Install macOS Sierra.app` kako bi se nadogradilo operativni sistem, koristeći `bless` alat. Komanda koja se koristi je sledeća:

```bash
/usr/sbin/bless -setBoot -folder /Volumes/Macintosh HD/macOS Install Data -bootefi /Volumes/Macintosh HD/macOS Install Data/boot.efi -options config="\macOS Install Data\com.apple.Boot" -label macOS Installer
```

Bezbednost ovog procesa može biti ugrožena ako napadač izmeni sliku nadogradnje (`InstallESD.dmg`) pre pokretanja. Strategija uključuje zamenu dinamičkog učitavača (dyld) zlonamernom verzijom (`libBaseIA.dylib`). Ova zamena rezultira izvršenjem koda napadača prilikom pokretanja instalera.

Kod napadača preuzima kontrolu tokom procesa nadogradnje, iskorišćavajući sistemsko poverenje u instalator. Napad se nastavlja izmenom slike `InstallESD.dmg` putem metode swizzling, posebno ciljajući metodu `extractBootBits`. To omogućava ubacivanje zlonamernog koda pre upotrebe slike diska.

Osim toga, unutar `InstallESD.dmg` nalazi se `BaseSystem.dmg`, koji služi kao korenski fajl sistem koda za nadogradnju. Ubacivanje dinamičke biblioteke u to omogućava zlonamernom kodu da radi unutar procesa koji može menjati fajlove na nivou operativnog sistema, značajno povećavajući mogućnost kompromitovanja sistema.

#### [systemmigrationd (2023)](https://www.youtube.com/watch?v=zxZesAN-TEk)

U ovom govoru sa [**DEF CON 31**](https://www.youtube.com/watch?v=zxZesAN-TEk), prikazano je kako **`systemmigrationd`** (koji može zaobići SIP) izvršava **bash** i **perl** skriptu, koja može biti zloupotrebljena putem okružnih promenljivih **`BASH_ENV`** i **`PERL5OPT`**.

### **com.apple.rootless.install**

{% hint style="danger" %}
Odobrenje **`com.apple.rootless.install`** omogućava zaobilaženje SIP-a
{% endhint %}

Odobrenje `com.apple.rootless.install` poznato je da zaobilazi System Integrity Protection (SIP) na macOS-u. Ovo je posebno pomenuto u vezi sa [**CVE-2022-26712**](https://jhftss.github.io/CVE-2022-26712-The-POC-For-SIP-Bypass-Is-Even-Tweetable/).

U ovom konkretnom slučaju, sistemski XPC servis smešten na lokaciji `/System/Library/PrivateFrameworks/ShoveService.framework/Versions/A/XPCServices/SystemShoveService.xpc` poseduje ovo odobrenje. To omogućava povezanim procesima da zaobiđu SIP ograničenja. Ovaj servis takođe ima metodu koja omogućava premestanje fajlova bez primene bilo kakvih sigurnosnih mera.

## Zapečaćene snimke sistema

Zapečaćene snimke sistema su funkcija koju je Apple uveo u **macOS Big Sur (macOS 11)** kao deo mehanizma **System Integrity Protection (SIP)** kako bi pružio dodatni nivo sigurnosti i stabilnosti sistema. To su suštinski samo za čitanje verzije volumena sistema.

Evo detaljnijeg pregleda:

1. **Nepromenljiv sistem**: Zapečaćene snimke sistema čine macOS sistemski volumen "nepromenljivim", što znači da ga nije moguće menjati. Ovo sprečava neovlaštene ili slučajne promene na sistemu koje bi mogle ugroziti sigurnost ili stabilnost sistema.
2. **Ažuriranje sistemskog softvera**: Kada instalirate ažuriranja ili nadogradnje macOS-a, macOS kreira novu snimku sistema. Pokretački volumen macOS-a zatim koristi **APFS (Apple File System)** da pređe na ovu novu snimku. Ceo proces primene ažuriranja postaje sigurniji i pouzdaniji jer sistem uvek može da se vrati na prethodnu snimku ako nešto pođe po zlu tokom ažuriranja.
3. **Razdvajanje podataka**: Uz koncept razdvajanja volumena podataka i sistema koji je uveden u macOS Catalina, funkcija zapečaćenih snimaka sistema obezbeđuje da se svi vaši podaci i podešavanja čuvaju na odvojenom "**Data**" volumenu. Ova razdvajanja čine vaše podatke nezavisnim od sistema, što pojednostavljuje proces ažuriranja sistema i poboljšava sigurnost sistema.

Zapamtite da ove snimke automatski upravlja macOS i ne zauzimaju dodatni prostor na disku, zahvaljujući mogućnostima deljenja prostora u APFS-u. Takođe je važno napomenuti da se ove snimke razlikuju od **Time Machine snimaka**, koji su korisnički dostupne rezervne kopije celog sistema.

### Provera snimaka

Komanda **`diskutil apfs list`** prikazuje **detalje o APFS volumenima** i njihovoj strukturi:

<pre><code>+-- Container disk3 966B902E-EDBA-4775-B743-CF97A0556A13
|   ====================================================
|   APFS Container Reference:     disk3
|   Size (Capacity Ceiling):      494384795648 B (494.4 GB)
|   Capacity In Use By Volumes:   219214536704 B (219.2 GB) (44.3% used)
|   Capacity Not Allocated:       275170258944 B (275.2 GB) (55.7% free)
|   |
|   +-&#x3C; Physical Store disk0s2 86D4B7EC-6FA5-4042-93A7-D3766A222EBE
|   |   -----------------------------------------------------------
|   |   APFS Physical Store Disk:   disk0s2
|   |   Size:                       494384795648 B (494.4 GB)
|   |
|   +-> Volume disk3s1 7A27E734-880F-4D91-A703-FB55861D49B7
|   |   ---------------------------------------------------
<strong>|   |   APFS Volume Disk (Role):   disk3s1 (System)
</strong>|   |   Name:                      Macintosh HD (Case-insensitive)
<strong>|   |   Mount Point:               /System/Volumes/Update/mnt1
</strong>|   |   Capacity Consumed:         12819210240 B (12.8 GB)
|   |   Sealed:                    Broken
|   |   FileVault:                 Yes (Unlocked)
|   |   Encrypted:                 No
|   |   |
|   |   Snapshot:                  FAA23E0C-791C-43FF-B0E7-0E1C0810AC61
|   |   Snapshot Disk:             disk3s1s1
<strong>|   |   Snapshot Mount Point:      /
</strong><strong>|   |   Snapshot Sealed:           Yes
</strong>[...]
+-> Volume disk3s5 281959B7-07A1-4940-BDDF-6419360F3327
|   ---------------------------------------------------
|   APFS Volume Disk (Role):   disk3s5 (Data)
|   Name:                      Macintosh HD - Data (Case-insensitive)
<strong>    |   Mount Point:               /System/Volumes/Data
</strong><strong>    |   Capacity Consumed:         412071784448 B (412.1 GB)
</strong>    |   Sealed:                    No
|   FileVault:                 Yes (Unlocked)
</code></pre>

U prethodnom izlazu mogu se videti **lokacije dostupne korisniku** montirane pod `/System/Volumes/Data`.

Osim toga, **snimak macOS sistemskog volumena** je montiran u `/` i **zapečaćen** (kriptografski potpisan od strane OS-a). Dakle, ako se zaobiđe SIP i izmeni, **OS se više neće pokrenuti**.

Takođe je moguće **proveriti da li je zapečaćenje omogućeno** pokretanjem:

```bash
csrutil authenticated-root status
Authenticated Root status: enabled
```

Osim toga, disk sa snimkom je takođe montiran kao **samo za čitanje**:

```
mount
/dev/disk3s1s1 on / (apfs, sealed, local, read-only, journaled)
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
