# Abusing Tokens

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite pristup **najnovijoj verziji PEASS ili preuzimanje HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) **Discord grupi**]\(https://discord.gg/hRep4RUj7f) ili **telegram grupi** ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove podnošenjem PR-ova na** [**hacktricks repozitorijumu**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijumu**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Tokeni

Ako **ne znate šta su Windows Access Tokeni** pročitajte ovu stranicu pre nego što nastavite:

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

**Možda ćete moći da eskalirate privilegije zloupotrebom tokena koje već imate**

### SeImpersonatePrivilege

Ovo je privilegija koju poseduje bilo koji proces koji omogućava impersonaciju (ali ne i kreiranje) bilo kog tokena, pod uslovom da se može dobiti ručka za njega. Privilegovani token može se dobiti od Windows servisa (DCOM) indukujući ga da izvrši NTLM autentikaciju protiv eksploatacije, čime se omogućava izvršenje procesa sa SISTEM privilegijama. Ova ranjivost može se iskoristiti korišćenjem različitih alata, poput [juicy-potato](https://github.com/ohpe/juicy-potato), [RogueWinRM](https://github.com/antonioCoco/RogueWinRM) (koji zahteva onemogućen winrm), [SweetPotato](https://github.com/CCob/SweetPotato) i [PrintSpoofer](https://github.com/itm4n/PrintSpoofer).

{% content-ref url="roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](roguepotato-and-printspoofer.md)
{% endcontent-ref %}

{% content-ref url="juicypotato.md" %}
[juicypotato.md](juicypotato.md)
{% endcontent-ref %}

### SeAssignPrimaryPrivilege

Veoma je slično **SeImpersonatePrivilege**, koristiće **isti metod** za dobijanje privilegovanog tokena.\
Onda, ova privilegija omogućava **dodelu primarnog tokena** novom/suspendovanom procesu. Sa privilegovanim impersonacionim tokenom možete izvesti primarni token (DuplicateTokenEx).\
Sa tokenom, možete kreirati **novi proces** sa 'CreateProcessAsUser' ili kreirati proces suspendovan i **postaviti token** (uopšteno, ne možete modifikovati primarni token pokrenutog procesa).

### SeTcbPrivilege

Ako ste omogućili ovaj token, možete koristiti **KERB\_S4U\_LOGON** da dobijete **impersonacioni token** za bilo kog drugog korisnika bez poznavanja akreditiva, **dodati proizvoljnu grupu** (administratori) tokenu, postaviti **nivo integriteta** tokena na "**srednji**" i dodeliti ovaj token **trenutnom thread-u** (SetThreadToken).

### SeBackupPrivilege

Sistem je primoran da **dodeli sve pristupe čitanju** kontrole bilo kog fajla (ograničeno na operacije čitanja) ovom privilegijom. Koristi se za **čitanje heševa lozinki lokalnih Administrator** naloga iz registra, nakon čega se mogu koristiti alati poput "**psexec**" ili "**wmicexec**" sa hešom (Pass-the-Hash tehnika). Međutim, ova tehnika ne uspeva pod dva uslova: kada je lokalni Administrator nalog onemogućen ili kada je politika koja uklanja administratorska prava od lokalnih administratora koji se povezuju na daljinu. Možete **zloupotrebiti ovu privilegiju** sa:

* [https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)
* [https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)
* prateći **IppSec** u [https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec](https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec)
* Ili kako je objašnjeno u odeljku **eskalacija privilegija sa Backup Operatorima** u:

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### SeRestorePrivilege

Dozvola za **pisanje pristupa** bilo kom sistemskom fajlu, bez obzira na Listu Kontrole Pristupa (ACL) fajla, obezbeđuje se ovom privilegijom. Otvora brojne mogućnosti za eskalaciju, uključujući mogućnost **modifikacije servisa**, izvođenje DLL Hijacking-a i postavljanje **debugera** putem Opcija Izvršenja Fajlova Slike među raznim drugim tehnikama.

### SeCreateTokenPrivilege

SeCreateTokenPrivilege je moćna dozvola, posebno korisna kada korisnik ima mogućnost impersonacije tokena, ali i u odsustvu SeImpersonatePrivilege. Ova sposobnost zavisi od mogućnosti impersonacije tokena koji predstavlja istog korisnika i čiji nivo integriteta ne premašuje nivo integriteta trenutnog procesa.

**Ključne tačke:**

* **Impersonacija bez SeImpersonatePrivilege:** Moguće je iskoristiti SeCreateTokenPrivilege za EoP impersonacijom tokena pod određenim uslovima.
* **Uslovi za Impersonaciju Tokena:** Uspešna impersonacija zahteva da ciljni token pripada istom korisniku i ima nivo integriteta koji je manji ili jednak nivou integriteta procesa koji pokušava impersonaciju.
* **Kreiranje i Modifikacija Impersonacionih Tokena:** Korisnici mogu kreirati impersonacioni token i unaprediti ga dodavanjem SID-a privilegovane grupe.

### SeLoadDriverPrivilege

Ova privilegija omogućava **učitavanje i isključivanje drajvera uređaja** sa kreiranjem unosa u registar sa specifičnim vrednostima za `ImagePath` i `Type`. Pošto je direktni pristup pisanju u `HKLM` (HKEY\_LOCAL\_MACHINE) ograničen, umesto toga mora se koristiti `HKCU` (HKEY\_CURRENT\_USER). Međutim, da bi se `HKCU` prepoznao od strane kernela za konfiguraciju drajvera, mora se pratiti određena putanja.

Ova putanja je `\Registry\User\<RID>\System\CurrentControlSet\Services\DriverName`, gde `<RID>` predstavlja Relativni Identifikator trenutnog korisnika. Unutar `HKCU`, mora se kreirati cela ova putanja, i postaviti dve vrednosti:

* `ImagePath`, što je putanja do binarnog fajla koji će se izvršiti
* `Type`, sa vrednošću `SERVICE_KERNEL_DRIVER` (`0x00000001`).

**Koraci koje treba pratiti:**

1. Pristupiti `HKCU` umesto `HKLM` zbog ograničenog pristupa pisanju.
2. Kreirati putanju `\Registry\User\<RID>\System\CurrentControlSet\Services\DriverName` unutar `HKCU`, gde `<RID>` predstavlja Relativni Identifikator trenutnog korisnika.
3. Postaviti `ImagePath` na putanju izvršenja binarnog fajla.
4. Dodeliti `Type` kao `SERVICE_KERNEL_DRIVER` (`0x00000001`).

```python
# Example Python code to set the registry values
import winreg as reg

# Define the path and values
path = r'Software\YourPath\System\CurrentControlSet\Services\DriverName' # Adjust 'YourPath' as needed
key = reg.OpenKey(reg.HKEY_CURRENT_USER, path, 0, reg.KEY_WRITE)
reg.SetValueEx(key, "ImagePath", 0, reg.REG_SZ, "path_to_binary")
reg.SetValueEx(key, "Type", 0, reg.REG_DWORD, 0x00000001)
reg.CloseKey(key)
```

Više načina za zloupotrebu ovlašćenja možete pronaći na [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege)

### SeTakeOwnershipPrivilege

Ovo je slično **SeRestorePrivilege**. Njegova osnovna funkcija omogućava procesu da **preuzme vlasništvo nad objektom**, zaobilazeći zahtev za eksplicitnim diskrecionim pristupom pružanjem prava pristupa WRITE\_OWNER. Proces uključuje prvo obezbeđivanje vlasništva nad odgovarajućim registarskim ključem u svrhu pisanja, a zatim izmenu DACL-a radi omogućavanja operacija pisanja.

```bash
takeown /f 'C:\some\file.txt' #Now the file is owned by you
icacls 'C:\some\file.txt' /grant <your_username>:F #Now you have full access
# Use this with files that might contain credentials such as
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software
%WINDIR%\repair\security
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
c:\inetpub\wwwwroot\web.config
```

### SeDebugPrivilege

Ova privilegija dozvoljava **debugovanje drugih procesa**, uključujući čitanje i pisanje u memoriju. Različite strategije za ubacivanje u memoriju, sposobne da izbegnu većinu antivirusnih i host intrusion prevention rešenja, mogu se koristiti sa ovom privilegijom.

#### Dump memorije

Možete koristiti [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) iz [SysInternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) da **zabeležite memoriju procesa**. Konkretno, ovo se može primeniti na proces **Local Security Authority Subsystem Service (**[**LSASS**](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service)**)**, koji je odgovoran za čuvanje korisničkih podataka nakon što se korisnik uspešno prijavi na sistem.

Zatim možete učitati ovaj dump u mimikatz-u da biste dobili lozinke:

```
mimikatz.exe
mimikatz # log
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```

#### RCE

Ako želite da dobijete `NT SYSTEM` shell, možete koristiti:

* [**SeDebugPrivilege-Exploit (C++)**](https://github.com/bruno-1337/SeDebugPrivilege-Exploit)
* [**SeDebugPrivilegePoC (C#)**](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC)
* [**psgetsys.ps1 (Powershell Script)**](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1)

```powershell
# Get the PID of a process running as NT SYSTEM
import-module psgetsys.ps1; [MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>)
```

## Provera privilegija

```
whoami /priv
```

**Tokeni koji se pojavljuju kao Onemogućeni** mogu se omogućiti, zapravo možete zloupotrebiti _Omogućene_ i _Onemogućene_ tokene.

### Omogući sve tokene

Ako imate onemogućene tokene, možete koristiti skriptu [**EnableAllTokenPrivs.ps1**](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) da omogućite sve tokene:

```powershell
.\EnableAllTokenPrivs.ps1
whoami /priv
```

Ili **skriptu** uključenu u ovom [**postu**](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/).

## Tabela

Potpuna lista privilegija tokena nalazi se na [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin), u nastavku je samo sažetak direktnih načina za iskorišćavanje privilegija radi dobijanja administratorske sesije ili čitanja osetljivih fajlova.

| Privilegija                | Uticaj      | Alat                   | Put izvršenja                                                                                                                                                                                                                                                                                                                                                     | Napomene                                                                                                                                                                                                                                                              |
| -------------------------- | ----------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`SeAssignPrimaryToken`** | _**Admin**_ | Alat treće strane      | _"Omogućava korisniku da se predstavlja tokenima i eskalira privilegije na nt sistem koristeći alate poput potato.exe, rottenpotato.exe i juicypotato.exe"_                                                                                                                                                                                                       | Hvala [Aurélien Chalot](https://twitter.com/Defte\_) na ažuriranju. Pokušaću da to preformulišem u nešto slično receptu uskoro.                                                                                                                                       |
| **`SeBackup`**             | **Pretnja** | _**Ugrađene komande**_ | Čitanje osetljivih fajlova sa `robocopy /b`                                                                                                                                                                                                                                                                                                                       | <p>- Može biti interesantno ako možete čitati %WINDIR%\MEMORY.DMP<br><br>- <code>SeBackupPrivilege</code> (i robocopy) nisu od pomoći kada su u pitanju otvoreni fajlovi.<br><br>- Robocopy zahteva i SeBackup i SeRestore da bi radio sa /b parametrom.</p>          |
| **`SeCreateToken`**        | _**Admin**_ | Alat treće strane      | Kreiranje proizvoljnog tokena uključujući lokalna administratorska prava sa `NtCreateToken`.                                                                                                                                                                                                                                                                      |                                                                                                                                                                                                                                                                       |
| **`SeDebug`**              | _**Admin**_ | **PowerShell**         | Dupliranje tokena `lsass.exe`.                                                                                                                                                                                                                                                                                                                                    | Skript možete pronaći na [FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1)                                                                                                                                             |
| **`SeLoadDriver`**         | _**Admin**_ | Alat treće strane      | <p>1. Učitajte problematični drajver jezgra poput <code>szkg64.sys</code><br>2. Iskoristite ranjivost drajvera<br><br>Alternativno, privilegija se može koristiti za isključivanje drajvera vezanih za bezbednost pomoću ugrađene komande <code>ftlMC</code>. npr.: <code>fltMC sysmondrv</code></p>                                                              | <p>1. Ranjivost <code>szkg64</code> je navedena kao <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732">CVE-2018-15732</a><br>2. Kod za eksploataciju <code>szkg64</code> je kreirao <a href="https://twitter.com/parvezghh">Parvez Anwar</a></p> |
| **`SeRestore`**            | _**Admin**_ | **PowerShell**         | <p>1. Pokrenite PowerShell/ISE sa prisutnom privilegijom SeRestore.<br>2. Omogućite privilegiju sa <a href="https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1">Enable-SeRestorePrivilege</a>).<br>3. Preimenujte utilman.exe u utilman.old<br>4. Preimenujte cmd.exe u utilman.exe<br>5. Zaključajte konzolu i pritisnite Win+U</p> | <p>Napad može biti otkriven od strane nekih AV softvera.</p><p>Alternativna metoda se oslanja na zamenu binarnih fajlova servisa smeštenih u "Program Files" koristeći istu privilegiju</p>                                                                           |
| **`SeTakeOwnership`**      | _**Admin**_ | _**Ugrađene komande**_ | <p>1. <code>takeown.exe /f "%windir%\system32"</code><br>2. <code>icalcs.exe "%windir%\system32" /grant "%username%":F</code><br>3. Preimenujte cmd.exe u utilman.exe<br>4. Zaključajte konzolu i pritisnite Win+U</p>                                                                                                                                            | <p>Napad može biti otkriven od strane nekih AV softvera.</p><p>Alternativna metoda se oslanja na zamenu binarnih fajlova servisa smeštenih u "Program Files" koristeći istu privilegiju.</p>                                                                          |
| **`SeTcb`**                | _**Admin**_ | Alat treće strane      | <p>Manipulacija tokenima radi uključivanja lokalnih administratorskih prava. Može zahtevati SeImpersonate.</p><p>Da se proveri.</p>                                                                                                                                                                                                                               |                                                                                                                                                                                                                                                                       |

## Reference

* Pogledajte ovu tabelu koja definiše Windows tokene: [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)
* Pogledajte [**ovaj rad**](https://github.com/hatRiot/token-priv/blob/master/abusing\_token\_eop\_1.0.txt) o eskalaciji privilegija pomoću tokena.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li videti **vašu kompaniju reklamiranu na HackTricks**? ili želite pristup **najnovijoj verziji PEASS ili preuzimanje HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
