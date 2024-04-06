# Windows Credentials Protections

## Zaštita pristupačnih podataka

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## WDigest

Protokol [WDigest](https://technet.microsoft.com/pt-pt/library/cc778868\(v=ws.10\).aspx?f=255\&MSPPError=-2147217396), koji je uveden sa Windows XP, dizajniran je za autentifikaciju putem HTTP protokola i **podrazumevano je omogućen na Windows XP-u do Windows 8.0 i Windows Server 2003 do Windows Server 2012**. Ova podrazumevana postavka rezultira **čuvanjem lozinki u tekstualnom formatu u LSASS-u** (Local Security Authority Subsystem Service). Napadač može koristiti alat Mimikatz da **izvuče ove pristupne podatke** izvršavanjem:

```bash
sekurlsa::wdigest
```

Da biste **uključili ili isključili ovu funkciju**, registarski ključevi _**UseLogonCredential**_ i _**Negotiate**_ unutar _**HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\SecurityProviders\WDigest**_ moraju biti postavljeni na "1". Ako ovi ključevi su **odsutni ili postavljeni na "0"**, WDigest je **onemogućen**:

```bash
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```

## LSA zaštita

Počevši od **Windows 8.1**, Microsoft je poboljšao sigurnost LSA da **blokira neovlašćeno čitanje memorije ili ubacivanje koda od strane nepouzdanih procesa**. Ovo poboljšanje otežava tipično funkcionisanje komandi poput `mimikatz.exe sekurlsa:logonpasswords`. Da biste **omogućili ovu poboljšanu zaštitu**, vrednost _**RunAsPPL**_ u _**HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\LSA**_ treba da se podesi na 1:

```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```

### Bypass

Moguće je zaobići ovu zaštitu koristeći Mimikatz drajver mimidrv.sys:

![](../../.gitbook/assets/mimidrv.png)

## Credential Guard

**Credential Guard**, funkcija ekskluzivna za **Windows 10 (Enterprise i Education izdanja)**, poboljšava sigurnost mašinskih akreditiva koristeći **Virtual Secure Mode (VSM)** i **Virtualization Based Security (VBS)**. Koristi proširenja virtualizacije CPU-a kako bi izolovala ključne procese unutar zaštićenog memorijskog prostora, izvan dosega glavnog operativnog sistema. Ova izolacija osigurava da čak ni kernel ne može pristupiti memoriji u VSM-u, efikasno štiteći akreditive od napada poput **pass-the-hash**. **Local Security Authority (LSA)** funkcioniše unutar ovog sigurnog okruženja kao trustlet, dok proces **LSASS** u glavnom OS-u deluje samo kao komunikator sa LSA-om VSM-a.

Po podrazumevanim podešavanjima, **Credential Guard** nije aktivan i zahteva ručno aktiviranje unutar organizacije. Kritično je za poboljšanje sigurnosti protiv alata poput **Mimikatz**, koji su ometeni u svojoj sposobnosti da izvuku akreditive. Međutim, ranjivosti se i dalje mogu iskoristiti dodavanjem prilagođenih **Security Support Provider (SSP)**-ova za hvatanje akreditiva u čistom tekstu tokom pokušaja prijavljivanja.

Da biste proverili status aktivacije **Credential Guard**-a, može se pregledati registarski ključ _**LsaCfgFlags**_ pod _**HKLM\System\CurrentControlSet\Control\LSA**_. Vrednost "**1**" ukazuje na aktivaciju sa **UEFI zaključavanjem**, "**2**" bez zaključavanja, a "**0**" označava da nije omogućeno. Ova provera registra, iako snažan pokazatelj, nije jedini korak za omogućavanje Credential Guard-a. Detaljno uputstvo i PowerShell skripta za omogućavanje ove funkcije dostupni su na internetu.

```powershell
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```

Za sveobuhvatno razumevanje i uputstva o omogućavanju **Credential Guard**-a u Windows 10 i njegovoj automatskoj aktivaciji u kompatibilnim sistemima **Windows 11 Enterprise i Education (verzija 22H2)**, posetite [Microsoft-ovu dokumentaciju](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage).

Dodatne detalje o implementaciji prilagođenih SSP-ova za hvatanje akreditacija možete pronaći u [ovom vodiču](../active-directory-methodology/custom-ssp.md).

## RDP RestrictedAdmin režim

**Windows 8.1 i Windows Server 2012 R2** su uveli nekoliko novih sigurnosnih funkcija, uključujući _**Restricted Admin režim za RDP**_. Ovaj režim je dizajniran da poboljša sigurnost tako što umanjuje rizike povezane sa [**pass the hash**](https://blog.ahasayen.com/pass-the-hash/) napadima.

Uobičajeno, kada se povežete sa udaljenim računarom putem RDP-a, vaše akreditacije se čuvaju na ciljnom računaru. Ovo predstavlja značajan sigurnosni rizik, posebno kada koristite naloge sa povišenim privilegijama. Međutim, sa uvođenjem _**Restricted Admin režima**_, ovaj rizik se značajno smanjuje.

Kada pokrenete RDP konekciju koristeći komandu **mstsc.exe /RestrictedAdmin**, autentifikacija na udaljenom računaru se vrši bez čuvanja vaših akreditacija na njemu. Ovaj pristup osigurava da, u slučaju infekcije malverom ili ako zlonamerni korisnik dobije pristup udaljenom serveru, vaše akreditacije neće biti kompromitovane, jer se ne čuvaju na serveru.

Važno je napomenuti da u **Restricted Admin režimu**, pokušaji pristupa mrežnim resursima iz RDP sesije neće koristiti vaše lične akreditacije; umesto toga, koristi se **identitet mašine**.

Ova funkcionalnost predstavlja značajan korak napred u obezbeđivanju udaljenih desktop konekcija i zaštiti osetljivih informacija od izlaganja u slučaju bezbednosnog propusta.

![](../../.gitbook/assets/ram.png)

Za detaljnije informacije posetite [ovaj izvor](https://blog.ahasayen.com/restricted-admin-mode-for-rdp/).

## Keširane akreditacije

Windows obezbeđuje **domenske akreditacije** putem **Local Security Authority (LSA)**, podržavajući procese prijavljivanja sa sigurnosnim protokolima kao što su **Kerberos** i **NTLM**. Ključna funkcionalnost Windows-a je mogućnost keširanja **poslednjih deset domenskih prijava** kako bi se osiguralo da korisnici i dalje mogu pristupiti svojim računarima čak i ako je **kontroler domena van mreže** - što je od koristi za korisnike laptopova koji su često van mreže svoje kompanije.

Broj keširanih prijava se može podešavati putem određenog **registarskog ključa ili grupe politika**. Da biste videli ili promenili ovu postavku, koristi se sledeća komanda:

```bash
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```

Pristup ovim keširanim akreditacijama strogo je kontrolisan, pri čemu samo **SYSTEM** nalog ima potrebne dozvole da ih pregleda. Administratori koji trebaju pristupiti ovim informacijama moraju to učiniti sa privilegijama korisnika SYSTEM. Akreditacije se čuvaju na lokaciji: `HKEY_LOCAL_MACHINE\SECURITY\Cache`

**Mimikatz** se može koristiti za izvlačenje ovih keširanih akreditacija pomoću komande `lsadump::cache`.

Za dalje detalje, originalni [izvor](http://juggernaut.wikidot.com/cached-credentials) pruža sveobuhvatne informacije.

## Zaštićeni korisnici

Članstvo u grupi **Zaštićeni korisnici** uvodi nekoliko sigurnosnih poboljšanja za korisnike, osiguravajući viši nivo zaštite od krađe i zloupotrebe akreditacija:

* **Delegiranje akreditacija (CredSSP)**: Čak i ako je postavka Grupe za politiku **Dozvoli delegiranje podrazumevanih akreditacija** omogućena, akreditacije zaštićenih korisnika neće biti keširane u obliku običnog teksta.
* **Windows Digest**: Počevši od **Windows 8.1 i Windows Server 2012 R2**, sistem neće keširati akreditacije zaštićenih korisnika u obliku običnog teksta, bez obzira na status Windows Digest-a.
* **NTLM**: Sistem neće keširati akreditacije zaštićenih korisnika u obliku običnog teksta ili NT jednosmernih funkcija (NTOWF).
* **Kerberos**: Za zaštićene korisnike, Kerberos autentifikacija neće generisati **DES** ili **RC4 ključeve**, niti će keširati akreditacije u obliku običnog teksta ili dugoročne ključeve izvan početnog dobijanja Ticket-Granting Ticket (TGT).
* **Offline prijava**: Za zaštićene korisnike neće biti kreiran keširani verifikator prilikom prijave ili otključavanja, što znači da offline prijava nije podržana za ove naloge.

Ove zaštite se aktiviraju čim se korisnik, koji je član grupe **Zaštićeni korisnici**, prijavi na uređaj. Ovo osigurava da su kritične sigurnosne mere na snazi radi zaštite od različitih metoda kompromitacije akreditacija.

Za detaljnije informacije, pogledajte zvaničnu [dokumentaciju](https://docs.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group).

**Tabela iz** [**dokumentacije**](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory)**.**

| Windows Server 2003 RTM | Windows Server 2003 SP1+ | <p>Windows Server 2012,<br>Windows Server 2008 R2,<br>Windows Server 2008</p> | Windows Server 2016          |
| ----------------------- | ------------------------ | ----------------------------------------------------------------------------- | ---------------------------- |
| Account Operators       | Account Operators        | Account Operators                                                             | Account Operators            |
| Administrator           | Administrator            | Administrator                                                                 | Administrator                |
| Administrators          | Administrators           | Administrators                                                                | Administrators               |
| Backup Operators        | Backup Operators         | Backup Operators                                                              | Backup Operators             |
| Cert Publishers         |                          |                                                                               |                              |
| Domain Admins           | Domain Admins            | Domain Admins                                                                 | Domain Admins                |
| Domain Controllers      | Domain Controllers       | Domain Controllers                                                            | Domain Controllers           |
| Enterprise Admins       | Enterprise Admins        | Enterprise Admins                                                             | Enterprise Admins            |
|                         |                          |                                                                               | Enterprise Key Admins        |
|                         |                          |                                                                               | Key Admins                   |
| Krbtgt                  | Krbtgt                   | Krbtgt                                                                        | Krbtgt                       |
| Print Operators         | Print Operators          | Print Operators                                                               | Print Operators              |
|                         |                          | Read-only Domain Controllers                                                  | Read-only Domain Controllers |
| Replicator              | Replicator               | Replicator                                                                    | Replicator                   |
| Schema Admins           | Schema Admins            | Schema Admins                                                                 | Schema Admins                |
| Server Operators        | Server Operators         | Server Operators                                                              | Server Operators             |

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
