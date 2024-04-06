# macOS Sandbox

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovne informacije

macOS Pesak (inicijalno nazvan Seatbelt) **ograničava aplikacije** koje se izvršavaju unutar peska na **dozvoljene akcije navedene u profilu peska** sa kojim se aplikacija izvršava. Ovo pomaže da se osigura da **aplikacija pristupa samo očekivanim resursima**.

Bilo koja aplikacija sa **ovlašćenjem** **`com.apple.security.app-sandbox`** će biti izvršena unutar peska. **Apple binarni fajlovi** se obično izvršavaju unutar peska i kako bi se objavili u **App Store-u**, **ovo ovlašćenje je obavezno**. Dakle, većina aplikacija će biti izvršena unutar peska.

Da bi se kontrolisalo šta proces može ili ne može da radi, **Pesak ima kuke** u svim **sistemskim pozivima** širom jezgra. **Zavisno** o **ovlašćenjima** aplikacije, Pesak će **dozvoliti** određene akcije.

Neki važni komponenti Peska su:

* **Kernel ekstenzija** `/System/Library/Extensions/Sandbox.kext`
* **Privatni okvir** `/System/Library/PrivateFrameworks/AppSandbox.framework`
* **Demon** koji se izvršava u korisničkom prostoru `/usr/libexec/sandboxd`
* **Kontejneri** `~/Library/Containers`

Unutar foldera kontejnera možete pronaći **folder za svaku aplikaciju koja se izvršava u pesku** sa imenom identifikatora paketa (bundle id):

```bash
ls -l ~/Library/Containers
total 0
drwx------@ 4 username  staff  128 May 23 20:20 com.apple.AMPArtworkAgent
drwx------@ 4 username  staff  128 May 23 20:13 com.apple.AMPDeviceDiscoveryAgent
drwx------@ 4 username  staff  128 Mar 24 18:03 com.apple.AVConference.Diagnostic
drwx------@ 4 username  staff  128 Mar 25 14:14 com.apple.Accessibility-Settings.extension
drwx------@ 4 username  staff  128 Mar 25 14:10 com.apple.ActionKit.BundledIntentHandler
[...]
```

Unutar svake fascikle sa identifikacijom paketa možete pronaći **plist** i **Data direktorijum** aplikacije:

```bash
cd /Users/username/Library/Containers/com.apple.Safari
ls -la
total 104
drwx------@   4 username  staff    128 Mar 24 18:08 .
drwx------  348 username  staff  11136 May 23 20:57 ..
-rw-r--r--    1 username  staff  50214 Mar 24 18:08 .com.apple.containermanagerd.metadata.plist
drwx------   13 username  staff    416 Mar 24 18:05 Data

ls -l Data
total 0
drwxr-xr-x@  8 username  staff   256 Mar 24 18:08 CloudKit
lrwxr-xr-x   1 username  staff    19 Mar 24 18:02 Desktop -> ../../../../Desktop
drwx------   2 username  staff    64 Mar 24 18:02 Documents
lrwxr-xr-x   1 username  staff    21 Mar 24 18:02 Downloads -> ../../../../Downloads
drwx------  35 username  staff  1120 Mar 24 18:08 Library
lrwxr-xr-x   1 username  staff    18 Mar 24 18:02 Movies -> ../../../../Movies
lrwxr-xr-x   1 username  staff    17 Mar 24 18:02 Music -> ../../../../Music
lrwxr-xr-x   1 username  staff    20 Mar 24 18:02 Pictures -> ../../../../Pictures
drwx------   2 username  staff    64 Mar 24 18:02 SystemData
drwx------   2 username  staff    64 Mar 24 18:02 tmp
```

{% hint style="danger" %}
Imajte na umu da čak i ako su simboličke veze tu da "pobegnu" iz Sandbox-a i pristupe drugim fasciklama, aplikacija i dalje mora **imati dozvole** da im pristupi. Ove dozvole se nalaze unutar **`.plist`** fajla.
{% endhint %}

```bash
# Get permissions
plutil -convert xml1 .com.apple.containermanagerd.metadata.plist -o -

# Binary sandbox profile
<key>SandboxProfileData</key>
<data>
AAAhAboBAAAAAAgAAABZAO4B5AHjBMkEQAUPBSsGPwsgASABHgEgASABHwEf...

# In this file you can find the entitlements:
<key>Entitlements</key>
<dict>
<key>com.apple.MobileAsset.PhishingImageClassifier2</key>
<true/>
<key>com.apple.accounts.appleaccount.fullaccess</key>
<true/>
<key>com.apple.appattest.spi</key>
<true/>
<key>keychain-access-groups</key>
<array>
<string>6N38VWS5BX.ru.keepcoder.Telegram</string>
<string>6N38VWS5BX.ru.keepcoder.TelegramShare</string>
</array>
[...]

# Some parameters
<key>Parameters</key>
<dict>
<key>_HOME</key>
<string>/Users/username</string>
<key>_UID</key>
<string>501</string>
<key>_USER</key>
<string>username</string>
[...]

# The paths it can access
<key>RedirectablePaths</key>
<array>
<string>/Users/username/Downloads</string>
<string>/Users/username/Documents</string>
<string>/Users/username/Library/Calendars</string>
<string>/Users/username/Desktop</string>
<key>RedirectedPaths</key>
<array/>
[...]
```

{% hint style="warning" %}
Sve što je kreirano/izmenjeno od strane aplikacije u pesku će dobiti **atribut karantina**. Ovo će sprečiti prostor peska da pokrene Gatekeeper ako aplikacija u pesku pokuša da izvrši nešto sa **`open`**.
{% endhint %}

### Profili peska

Profil peska su konfiguracioni fajlovi koji ukazuju šta će biti **dozvoljeno/zabranjeno** u tom **pesku**. Koristi se jezik profila peska (SBPL), koji koristi [**Scheme**](https://en.wikipedia.org/wiki/Scheme\_\(programming\_language\)) programski jezik.

Ovde možete pronaći primer:

```scheme
(version 1) ; First you get the version

(deny default) ; Then you shuold indicate the default action when no rule applies

(allow network*) ; You can use wildcards and allow everything

(allow file-read* ; You can specify where to apply the rule
(subpath "/Users/username/")
(literal "/tmp/afile")
(regex #"^/private/etc/.*")
)

(allow mach-lookup
(global-name "com.apple.analyticsd")
)
```

{% hint style="success" %}
Pogledajte ovaj [**istraživački rad**](https://reverse.put.as/2011/09/14/apple-sandbox-guide-v1-0/) **da biste proverili više akcija koje mogu biti dozvoljene ili odbijene.**
{% endhint %}

Važne **sistemske usluge** takođe se izvršavaju unutar svojih sopstvenih prilagođenih **sandbox-ova**, kao što je `mdnsresponder` usluga. Ove prilagođene **sandbox profile** možete videti unutar:

* **`/usr/share/sandbox`**
* **`/System/Library/Sandbox/Profiles`**
* Ostali sandbox profili mogu se proveriti na [https://github.com/s7ephen/OSX-Sandbox--Seatbelt--Profiles](https://github.com/s7ephen/OSX-Sandbox--Seatbelt--Profiles).

Aplikacije iz **App Store-a** koriste profil **`/System/Library/Sandbox/Profiles/application.sb`**. U ovom profilu možete proveriti kako ovlašćenja poput **`com.apple.security.network.server`** omogućavaju procesu korišćenje mreže.

SIP je Sandbox profil nazvan platform\_profile u /System/Library/Sandbox/rootless.conf

### Primeri Sandbox Profila

Da biste pokrenuli aplikaciju sa **određenim sandbox profilom**, možete koristiti:

```bash
sandbox-exec -f example.sb /Path/To/The/Application
```

{% code title="touch.sb" %}
````plaintext
(version 1)
(deny default)
(allow file-read-metadata)
(allow file-write-metadata)
(allow file-read-data (literal "/private/var/tmp/"))
(allow file-write-data (literal "/private/var/tmp/"))
(allow file-read-data (regex #"^/private/var/folders/[^/]+/[^/]+/[C,T]/"))
(allow file-write-data (regex #"^/private/var/folders/[^/]+/[^/]+/[C,T]/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/T/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/T/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/T/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/T/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/T/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/T/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/T/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/T/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/C/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/C/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/C/C/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/C/C/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/T/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/T/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/T/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/T/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/T/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/C/T/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/T/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/C/T/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/C/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/T/T/C/T/"))
(allow file-read-data (literal "/private/var/folders/[^/]+/[^/]+/[^/]+/C/T/C/T/"))
(allow file-write-data (literal "/private/var/folders/[^/]+/[^/]+/[^/
```scheme
(version 1)
(deny default)
(allow file* (literal "/tmp/hacktricks.txt"))
````
{% endcode %}

```bash
# This will fail because default is denied, so it cannot execute touch
sandbox-exec -f touch.sb touch /tmp/hacktricks.txt
# Check logs
log show --style syslog --predicate 'eventMessage contains[c] "sandbox"' --last 30s
[...]
2023-05-26 13:42:44.136082+0200  localhost kernel[0]: (Sandbox) Sandbox: sandbox-exec(41398) deny(1) process-exec* /usr/bin/touch
2023-05-26 13:42:44.136100+0200  localhost kernel[0]: (Sandbox) Sandbox: sandbox-exec(41398) deny(1) file-read-metadata /usr/bin/touch
2023-05-26 13:42:44.136321+0200  localhost kernel[0]: (Sandbox) Sandbox: sandbox-exec(41398) deny(1) file-read-metadata /var
2023-05-26 13:42:52.701382+0200  localhost kernel[0]: (Sandbox) 5 duplicate reports for Sandbox: sandbox-exec(41398) deny(1) file-read-metadata /var
[...]
```

{% code title="touch2.sb" %}
```
```
{% endcode %}

```scheme
(version 1)
(deny default)
(allow file* (literal "/tmp/hacktricks.txt"))
(allow process* (literal "/usr/bin/touch"))
; This will also fail because:
; 2023-05-26 13:44:59.840002+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-metadata /usr/bin/touch
; 2023-05-26 13:44:59.840016+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-data /usr/bin/touch
; 2023-05-26 13:44:59.840028+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-data /usr/bin
; 2023-05-26 13:44:59.840034+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-metadata /usr/lib/dyld
; 2023-05-26 13:44:59.840050+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) sysctl-read kern.bootargs
; 2023-05-26 13:44:59.840061+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-data /
```

{% code title="touch3.sb" %}
```scheme
(version 1)
(deny default)
(allow file* (literal "/private/tmp/hacktricks.txt"))
(allow process* (literal "/usr/bin/touch"))
(allow file-read-data (literal "/"))
; This one will work
```
{% endcode %}

{% hint style="info" %}
Napomena da **Apple-ov** **softver** koji se pokreće na **Windowsu** **nema dodatne sigurnosne mere**, kao što je sandboxing aplikacija.
{% endhint %}

Primeri zaobilaženja:

* [https://lapcatsoftware.com/articles/sandbox-escape.html](https://lapcatsoftware.com/articles/sandbox-escape.html)
* [https://desi-jarvis.medium.com/office365-macos-sandbox-escape-fcce4fa4123c](https://desi-jarvis.medium.com/office365-macos-sandbox-escape-fcce4fa4123c) (mogu da pišu datoteke van sandboxa čije ime počinje sa `~$`).

### Profili macOS Sandbox-a

macOS čuva sistemske profile sandbox-a na dve lokacije: **/usr/share/sandbox/** i **/System/Library/Sandbox/Profiles**.

Ako treća strana aplikacija ima _**com.apple.security.app-sandbox**_ privilegiju, sistem primenjuje profil **/System/Library/Sandbox/Profiles/application.sb** na taj proces.

### **iOS Sandbox Profil**

Podrazumevani profil se naziva **container** i nemamo SBPL tekstualnu reprezentaciju. U memoriji, ovaj sandbox je predstavljen kao binarno stablo Dozvoli/Odbij za svaku dozvolu iz sandbox-a.

### Debugiranje i zaobilaženje Sandbox-a

Na macOS-u, za razliku od iOS-a gde su procesi od početka sandbox-ovani od strane jezgra, **procesi moraju sami da se odluče za sandbox**. To znači da na macOS-u, proces nije ograničen sandbox-om sve dok aktivno ne odluči da uđe u njega.

Procesi automatski dobijaju Sandbox kada se pokrenu iz korisničkog prostora ako imaju privilegiju: `com.apple.security.app-sandbox`. Za detaljno objašnjenje ovog procesa pogledajte:

{% content-ref url="macos-sandbox-debug-and-bypass/" %}
[macos-sandbox-debug-and-bypass](macos-sandbox-debug-and-bypass/)
{% endcontent-ref %}

### **Provera privilegija PID-a**

[**Prema ovome**](https://www.youtube.com/watch?v=mG715HcDgO8\&t=3011s), **`sandbox_check`** (to je `__mac_syscall`), može proveriti **da li je operacija dozvoljena ili ne** od strane sandbox-a za određeni PID.

Alatka [**sbtool**](http://newosxbook.com/src.jl?tree=listings\&file=sbtool.c) može proveriti da li PID može izvršiti određenu radnju:

```bash
sbtool <pid> mach #Check mac-ports (got from launchd with an api)
sbtool <pid> file /tmp #Check file access
sbtool <pid> inspect #Gives you an explaination of the sandbox profile
sbtool <pid> all
```

### Prilagođeni SBPL u aplikacijama App Store-a

Moguće je da kompanije svoje aplikacije pokreću **sa prilagođenim Sandbox profilima** (umesto sa podrazumevanim). Za to je potrebno koristiti privilegiju **`com.apple.security.temporary-exception.sbpl`** koju mora odobriti Apple.

Moguće je proveriti definiciju ove privilegije u **`/System/Library/Sandbox/Profiles/application.sb:`**

```scheme
(sandbox-array-entitlement
"com.apple.security.temporary-exception.sbpl"
(lambda (string)
(let* ((port (open-input-string string)) (sbpl (read port)))
(with-transparent-redirection (eval sbpl)))))
```

Ovo će **proceniti string nakon ovog ovlašćenja** kao Sandbox profil.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
