# Active Directory Methodology

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovni pregled

**Active Directory** služi kao temeljna tehnologija koja omogućava **mrežnim administratorima** efikasno kreiranje i upravljanje **domenima**, **korisnicima** i **objektima** unutar mreže. Projektovan je da se skalira, olakšavajući organizaciju velikog broja korisnika u upravljive **grupe** i **podgrupe**, dok istovremeno kontroliše **pristupna prava** na različitim nivoima.

Struktura **Active Directory**-ja se sastoji od tri osnovna sloja: **domeni**, **stablo** i **šume**. **Domen** obuhvata kolekciju objekata, kao što su **korisnici** ili **uređaji**, koji dele zajedničku bazu podataka. **Stabla** su grupe ovih domena povezanih zajedničkom strukturom, a **šuma** predstavlja kolekciju više stabala, povezanih putem **poverenih odnosa**, čineći najviši sloj organizacione strukture. Specifična **prava pristupa** i **komunikacije** mogu biti određena na svakom od ovih nivoa.

Ključni koncepti unutar **Active Directory**-ja uključuju:

1. **Direktorijum** - Sadrži sve informacije koje se odnose na objekte Active Directory-ja.
2. **Objekat** - Označava entitete unutar direktorijuma, uključujući **korisnike**, **grupe** ili **deljene fascikle**.
3. **Domen** - Služi kao kontejner za objekte direktorijuma, sa mogućnošću da više domena koegzistira unutar **šume**, pri čemu svaki održava sopstvenu kolekciju objekata.
4. **Stablo** - Grupisanje domena koji dele zajednički korenski domen.
5. **Šuma** - Vrhunac organizacione strukture u Active Directory-ju, sastoji se od nekoliko stabala sa **poverenim odnosima** među njima.

**Active Directory Domain Services (AD DS)** obuhvata niz usluga koje su ključne za centralizovano upravljanje i komunikaciju unutar mreže. Ove usluge obuhvataju:

1. **Usluge domena** - Centralizuje skladištenje podataka i upravlja interakcijama između **korisnika** i **domena**, uključujući **autentifikaciju** i **pretragu**.
2. **Usluge sertifikata** - Nadgleda kreiranje, distribuciju i upravljanje sigurnim **digitalnim sertifikatima**.
3. **Usluge lakog direktorijuma** - Podržava aplikacije sa omogućenim direktorijumom putem **LDAP protokola**.
4. **Usluge federacije direktorijuma** - Pruža mogućnosti **jednokratne prijave** za autentifikaciju korisnika na više veb aplikacija u jednoj sesiji.
5. **Upravljanje pravima** - Pomaže u zaštiti autorskih materijala regulisanjem neovlašćene distribucije i upotrebe.
6. **DNS usluga** - Ključna za razrešavanje **imenovanja domena**.

Za detaljnije objašnjenje pogledajte: [**TechTerms - Definicija Active Directory-ja**](https://techterms.com/definition/active\_directory)

### **Kerberos autentifikacija**

Da biste naučili kako **napasti AD**, morate **dobro razumeti proces Kerberos autentifikacije**.\
[**Pročitajte ovu stranicu ako još uvek ne znate kako to funkcioniše.**](kerberos-authentication.md)

## Cheat Sheet

Možete posetiti [https://wadcoms.github.io/](https://wadcoms.github.io) da biste brzo videli koje komande možete pokrenuti da biste nabrojali/iskoristili AD.

## Rekonstrukcija Active Directory-ja (bez akreditacija/sesija)

Ako imate pristup okruženju AD-a, ali nemate akreditacije/sesije, možete:

* **Testirajte mrežu:**
* Skenirajte mrežu, pronađite mašine i otvorene portove i pokušajte **iskoristiti ranjivosti** ili **izvući akreditacije** sa njih (na primer, [štampači mogu biti veoma interesantne mete](ad-information-in-printers.md)).
* Nabrojavanje DNS-a može pružiti informacije o ključnim serverima u domenu kao što su veb, štampači, deljenje, VPN, mediji, itd.
* `gobuster dns -d domain.local -t 25 -w /opt/Seclist/Discovery/DNS/subdomain-top2000.txt`
* Pogledajte opštu [**Metodologiju testiranja penetracije**](../../generic-methodologies-and-resources/pentesting-methodology.md) da biste pronašli više informacija o tome kako to uraditi.
* **Proverite pristup nuli i gostu na smb uslugama** (ovo neće raditi na modernim verzijama Windows-a):
* `enum4linux -a -u "" -p "" <DC IP> && enum4linux -a -u "guest" -p "" <DC IP>`
* `smbmap -u "" -p "" -P 445 -H <DC IP> && smbmap -u "guest" -p "" -P 445 -H <DC IP>`
* `smbclient -U '%' -L //<DC IP> && smbclient -U 'guest%' -L //`
* Detaljniji vodič o tome kako nabrojati SMB server možete pronaći ovde:

{% content-ref url="../../network-services-pentesting/pentesting-smb/" %}
[pentesting-smb](../../network-services-pentesting/pentesting-smb/)
{% endcontent-ref %}

* **Nabrojavanje LDAP-a**
* `nmap -n -sV --script "ldap* and not brute" -p 389 <DC IP>`
* Detaljniji vodič o tome kako nabrojati LDAP možete pronaći ovde (posebno **obratite pažnju na anonimni pristup**):

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

* \*\*Trovanje mreže

### Enumeracija korisnika

* **Anonimna SMB/LDAP enumeracija:** Pogledajte stranice [**pentesting SMB**](../../network-services-pentesting/pentesting-smb/) i [**pentesting LDAP**](../../network-services-pentesting/pentesting-ldap.md).
* **Kerbrute enumeracija**: Kada se zahteva **nevažeće korisničko ime**, server će odgovoriti koristeći **Kerberos grešku** sa kodom _KRB5KDC\_ERR\_C\_PRINCIPAL\_UNKNOWN_, što nam omogućava da utvrdimo da je korisničko ime nevažeće. **Validna korisnička imena** će izazvati ili **TGT u AS-REP** odgovoru ili grešku _KRB5KDC\_ERR\_PREAUTH\_REQUIRED_, što ukazuje da je korisniku potrebno izvršiti pre-authentication.

```bash
./kerbrute_linux_amd64 userenum -d lab.ropnop.com --dc 10.10.10.10 usernames.txt #From https://github.com/ropnop/kerbrute/releases

nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='DOMAIN'" <IP>
Nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm='<domain>',userdb=/root/Desktop/usernames.txt <IP>

msf> use auxiliary/gather/kerberos_enumusers

crackmapexec smb dominio.es  -u '' -p '' --users | awk '{print $4}' | uniq
```

* **OWA (Outlook Web Access) Server**

Ako pronađete jedan od ovih servera u mreži, takođe možete izvršiti **enumeraciju korisnika protiv njega**. Na primer, možete koristiti alat [**MailSniper**](https://github.com/dafthack/MailSniper):

```bash
ipmo C:\Tools\MailSniper\MailSniper.ps1
# Get info about the domain
Invoke-DomainHarvestOWA -ExchHostname [ip]
# Enumerate valid users from a list of potential usernames
Invoke-UsernameHarvestOWA -ExchHostname [ip] -Domain [domain] -UserList .\possible-usernames.txt -OutFile valid.txt
# Password spraying
Invoke-PasswordSprayOWA -ExchHostname [ip] -UserList .\valid.txt -Password Summer2021
# Get addresses list from the compromised mail
Get-GlobalAddressList -ExchHostname [ip] -UserName [domain]\[username] -Password Summer2021 -OutFile gal.txt
```

{% hint style="warning" %}
Možete pronaći liste korisničkih imena u [**ovom github repozitorijumu**](https://github.com/danielmiessler/SecLists/tree/master/Usernames/Names) \*\*\*\* i ovom ([**statistically-likely-usernames**](https://github.com/insidetrust/statistically-likely-usernames)).

Međutim, trebali biste imati **ime ljudi koji rade u kompaniji** iz koraka izviđanja koji ste trebali obaviti pre ovoga. Sa imenom i prezimenom možete koristiti skriptu [**namemash.py**](https://gist.github.com/superkojiman/11076951) da generišete potencijalno validna korisnička imena.
{% endhint %}

### Poznavanje jednog ili više korisničkih imena

Dobro, znate da već imate validno korisničko ime, ali nemate lozinke... Zatim pokušajte:

* [**ASREPRoast**](asreproast.md): Ako korisnik **nema** atribut _DONT\_REQ\_PREAUTH_, možete **zahtevati AS\_REP poruku** za tog korisnika koja će sadržati neke podatke šifrovane izvedenicom korisnikove lozinke.
* [**Password Spraying**](password-spraying.md): Pokušajte sa naj**češćim lozinkama** za svakog otkrivenog korisnika, možda neki korisnik koristi lošu lozinku (imajte na umu politiku lozinke!).
* Imajte na umu da takođe možete **izvršiti prskanje OWA servera** da biste pokušali da pristupite korisničkim mail serverima.

{% content-ref url="password-spraying.md" %}
[password-spraying.md](password-spraying.md)
{% endcontent-ref %}

### LLMNR/NBT-NS trovanje

Možda ćete moći **dobiti** neke izazovne **hešove** za pucanje **trovanjem** nekih protokola **mreže**:

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

### NTML Relay

Ako ste uspeli da nabrojite aktivni direktorijum, imaćete **više emailova i bolje razumevanje mreže**. Možda ćete moći da izvršite **napade NTML preusmeravanja** \*\*\*\* da biste pristupili AD okruženju.

### Krađa NTML podataka

Ako možete **pristupiti drugim računarima ili deljenim resursima** koristeći **null ili gost korisnika**, možete **postaviti datoteke** (poput SCF datoteke) koje će, ako se nekako pristupi, **pokrenuti NTML autentifikaciju protiv vas** kako biste mogli da **ukradete** NTML izazov i pokušate ga puknuti:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

## Nabrojavanje aktivnog direktorijuma SA akreditacijama/sesijom

Za ovu fazu morate **kompromitovati akreditacije ili sesiju važećeg domenskog naloga**. Ako imate neke važeće akreditacije ili shell kao domenski korisnik, **trebali biste zapamtiti da su opcije date ranije i dalje opcije za kompromitovanje drugih korisnika**.

Pre nego što započnete autentifikovano nabrojavanje, trebali biste znati šta je **Kerberos problem dvostrukog skoka**.

{% content-ref url="kerberos-double-hop-problem.md" %}
[kerberos-double-hop-problem.md](kerberos-double-hop-problem.md)
{% endcontent-ref %}

### Nabrojavanje

Kompromitacija naloga je **veliki korak za početak kompromitovanja celog domena**, jer ćete biti u mogućnosti da započnete **nabrojavanje aktivnog direktorijuma**:

Što se tiče [**ASREPRoast**](asreproast.md), sada možete pronaći svakog mogućeg ranjivog korisnika, a što se tiče [**Password Spraying**](password-spraying.md), možete dobiti **listu svih korisničkih imena** i isprobati lozinku kompromitovanog naloga, prazne lozinke i nove obećavajuće lozinke.

* Možete koristiti [**CMD za osnovno izviđanje**](../basic-cmd-for-pentesters.md#domain-info)
* Takođe možete koristiti [**powershell za izviđanje**](../basic-powershell-for-pentesters/) što će biti prikrivenije
* Možete takođe [**koristiti powerview**](../basic-powershell-for-pentesters/powerview.md) da biste izvukli detaljnije informacije
* Još jedan neverovatan alat za izviđanje u aktivnom direktorijumu je [**BloodHound**](bloodhound.md). Nije baš prikriven (zavisno o metodama prikupljanja koje koristite), ali **ako vam to nije važno**, svakako ga isprobajte. Pronađite gde korisnici mogu RDP, pronađite put do drugih grupa itd.
* **Drugi automatizovani alati za nabrojavanje AD su:** [**AD Explorer**](bloodhound.md#ad-explorer)**,** [**ADRecon**](bloodhound.md#adrecon)**,** [**Group3r**](bloodhound.md#group3r)**,** [**PingCastle**](bloodhound.md#pingcastle)**.**
* [**DNS zapisi AD**](ad-dns-records.md) jer mogu sadržati zanimljive informacije.
* Alat sa **grafičkim korisničkim interfejsom** koji možete koristiti za nabrojavanje direktorijuma je **AdExplorer.exe** iz **SysInternal** Suite.
* Takođe možete pretraživati LDAP bazu podataka sa **ldapsearch** da biste pronašli akreditacije u poljima _userPassword_ & _unixUserPassword_, ili čak u polju _Description_. cf. [Lozinka u komentaru AD korisnika na PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#password-in-ad-user-comment) za druge metode.
* Ako koristite **Linux**, takođe možete nabrojati domen koristeći [**pywerview**](https://github.com/the-useless-one/pywerview).
* Takođe možete isprobati automatizovane alate kao:
* [**tomcarver16/ADSearch**](https://github.com/tomcarver16/ADSearch)
* [**61106960/adPEAS**](https://github.com/61106960/adPEAS)
* **Izdvajanje svih korisnika domena**

Veoma je jednostavno dobiti sva korisnička imena domena iz Windowsa (`net user /domain`, `Get-DomainUser` ili `wmic useraccount get name,sid`). Na Linuxu možete koristiti: `GetADUsers.py -all -dc-ip 10.10.10.110 domain.com/username` ili `enum4linux -a -u "user" -p "password" <DC IP>`

> Iako ova sekcija Nabrojavanje izgleda mala, to je najvažniji deo od svih. Pristupite linkovima (posebno onom za cmd, powershell, powerview i BloodHound), naučite kako da nabrojite domen i vežbajte dok se ne osećate sigurno. Tokom procene, ovo će biti ključni trenutak za pronalaženje puta do DA ili odlučivanje da ništa ne može biti urađeno.

### Kerberoast

Kerberoasting uključuje dobijanje **TGS karata** koje koriste usluge povezane sa korisničkim nalozima i pucanje njihove šifrovanja - koje se zasniva na korisničkim lozinkama - **offline**.

Više o tome u:

{% content-ref url="kerberoast.md" %}
[kerberoast.md](kerberoast.md)
{% endcontent-ref %}

### Udaljena veza (RDP, SSH, FTP, Win-RM, itd)

Kada ste dobili neke akreditive, možete proveriti da li imate pristup bilo kojem **računaru**. Za to možete koristiti **CrackMapExec** da biste pokušali da se povežete na nekoliko servera sa različitim protokolima, u skladu sa skeniranjem portova.

### Lokalno eskaliranje privilegija

Ako imate kompromitovane akreditive ili sesiju kao običan korisnik domena i imate **pristup** sa ovim korisnikom na **bilo kojem računaru u domenu**, trebali biste pokušati da pronađete način da **lokalno eskalirate privilegije i preuzmete akreditive**. To je zato što samo sa lokalnim administratorskim privilegijama možete **izvući heševe drugih korisnika** iz memorije (LSASS) i lokalno (SAM).

U ovom priručniku postoji cela stranica o [**lokalnom eskaliranju privilegija u Windowsu**](../windows-local-privilege-escalation/) i [**checklista**](../checklist-windows-privilege-escalation.md). Takođe, ne zaboravite da koristite [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite).

### Trenutne sesijske karte

Vrlo je **malo verovatno** da ćete pronaći **karte** u trenutnom korisniku koje vam daju dozvolu za pristup **neočekivanim resursima**, ali možete proveriti:

```bash
## List all tickets (if not admin, only current user tickets)
.\Rubeus.exe triage
## Dump the interesting one by luid
.\Rubeus.exe dump /service:krbtgt /luid:<luid> /nowrap
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))
```

### NTML Relay

Ako ste uspeli da nabrojite aktivni direktorijum, imaćete **više emailova i bolje razumevanje mreže**. Možda ćete moći da izvršite napade **NTML preusmeravanja**.

### **Traženje podataka za prijavu u deljenim računarima**

Sada kada imate neke osnovne podatke za prijavu, trebali biste proveriti da li možete **pronaći** bilo **koje zanimljive datoteke koje se dele unutar AD**. To možete uraditi ručno, ali to je veoma dosadan i ponavljajući zadatak (posebno ako pronađete stotine dokumenata koje treba proveriti).

[**Pratite ovaj link da biste saznali o alatima koje možete koristiti.**](../../network-services-pentesting/pentesting-smb/#domain-shared-folders-search)

### Krađa NTLM podataka za prijavu

Ako možete **pristupiti drugim računarima ili deljenim resursima**, možete **postaviti datoteke** (poput SCF datoteke) koje će, ako se nekako pristupi, **inicirati NTML autentifikaciju prema vama**, tako da možete **ukrasti** NTLM izazov kako biste ga probili:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

### CVE-2021-1675/CVE-2021-34527 PrintNightmare

Ova ranjivost omogućava svakom autentifikovanom korisniku da **ugrozi kontroler domena**.

{% content-ref url="printnightmare.md" %}
[printnightmare.md](printnightmare.md)
{% endcontent-ref %}

## Eskalacija privilegija na Active Directory SA privilegovanim podacima/sesijom

**Za sledeće tehnike, običan korisnik domena nije dovoljan, potrebne su vam posebne privilegije/podaci za prijavu kako biste izvršili ove napade.**

### Izvlačenje heša

Nadamo se da ste uspeli da **ugrozite neki lokalni administratorski** nalog koristeći [AsRepRoast](asreproast.md), [Password Spraying](password-spraying.md), [Kerberoast](kerberoast.md), [Responder](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md) uključujući preusmeravanje, [EvilSSDP](../../generic-methodologies-and-resources/pentesting-network/spoofing-ssdp-and-upnp-devices.md), [escalating privileges locally](../windows-local-privilege-escalation/).\
Onda je vreme da izvučete sve heševe iz memorije i lokalno.\
[**Pročitajte ovu stranicu o različitim načinima dobijanja heševa.**](https://github.com/carlospolop/hacktricks/blob/rs/windows-hardening/active-directory-methodology/broken-reference/README.md)

### Pass the Hash

**Kada imate heš korisnika**, možete ga koristiti da se **predstavljate** kao taj korisnik.\
Treba vam neki **alat** koji će **izvršiti** NTLM autentifikaciju koristeći **taj heš**, **ili** možete kreirati novu **sesiju za prijavu** i **ubaciti** taj **heš** unutar **LSASS**, tako da kada se izvrši bilo koja **NTLM autentifikacija**, taj **heš će biti korišćen**. Poslednja opcija je ono što radi mimikatz.\
[**Pročitajte ovu stranicu za više informacija.**](../ntlm/#pass-the-hash)

### Over Pass the Hash/Pass the Key

Ovaj napad ima za cilj **korišćenje NTLM heša korisnika za zahtevanje Kerberos tiketa**, kao alternativu uobičajenom Pass The Hash preko NTLM protokola. Stoga, ovo može biti posebno **korisno u mrežama gde je NTLM protokol onemogućen**, a dozvoljen je samo Kerberos kao protokol za autentifikaciju.

{% content-ref url="over-pass-the-hash-pass-the-key.md" %}
[over-pass-the-hash-pass-the-key.md](over-pass-the-hash-pass-the-key.md)
{% endcontent-ref %}

### Pass the Ticket

U metodi napada **Pass The Ticket (PTT)**, napadači **kradu autentifikacioni tiket korisnika** umesto njihove lozinke ili heš vrednosti. Taj ukradeni tiket se zatim koristi za **predstavljanje korisnika**, sticanje neovlašćenog pristupa resursima i uslugama unutar mreže.

{% content-ref url="pass-the-ticket.md" %}
[pass-the-ticket.md](pass-the-ticket.md)
{% endcontent-ref %}

### Ponovna upotreba podataka za prijavu

Ako imate **heš** ili **lozinku** lokalnog **administratora**, trebali biste pokušati da se **prijavite lokalno** na druge **računare** sa tim podacima.

```bash
# Local Auth Spray (once you found some local admin pass or hash)
## --local-auth flag indicate to only try 1 time per machine
crackmapexec smb --local-auth 10.10.10.10/23 -u administrator -H 10298e182387f9cab376ecd08491764a0 | grep +
```

{% hint style="warning" %}
Napomena da je ovo prilično **buka** i **LAPS** bi to **umirio**.
{% endhint %}

### Zloupotreba MSSQL-a i pouzdanih veza

Ako korisnik ima privilegije za **pristup MSSQL instancama**, može ih koristiti za **izvršavanje komandi** na MSSQL hostu (ako se pokreće kao SA), **ukrasti** NetNTLM **hash** ili čak izvršiti **preusmeravanje napada**.\
Takođe, ako je MSSQL instanca pouzdana (veza baze podataka) od strane druge MSSQL instance. Ako korisnik ima privilegije nad pouzdanom bazom podataka, moći će **koristiti odnos poverenja za izvršavanje upita i u drugoj instanci**. Ove veze mogu biti povezane i korisnik na kraju može pronaći pogrešno konfigurisanu bazu podataka u kojoj može izvršavati komande.\
**Veze između baza podataka rade čak i preko poverenja šuma.**

{% content-ref url="abusing-ad-mssql.md" %}
[abusing-ad-mssql.md](abusing-ad-mssql.md)
{% endcontent-ref %}

### Neograničeno preusmeravanje

Ako pronađete bilo koji objekat računara sa atributom [ADS\_UF\_TRUSTED\_FOR\_DELEGATION](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx) i imate privilegije domena na računaru, moći ćete izvući TGT-ove iz memorije svih korisnika koji se prijavljuju na računar.\
Dakle, ako se **Administrator domena prijavi na računar**, moći ćete izvući njegov TGT i preuzeti njegov identitet koristeći [Pass the Ticket](pass-the-ticket.md).\
Zahvaljujući ograničenom preusmeravanju, čak možete **automatski kompromitovati Print Server** (nadam se da će to biti DC).

{% content-ref url="unconstrained-delegation.md" %}
[unconstrained-delegation.md](unconstrained-delegation.md)
{% endcontent-ref %}

### Ograničeno preusmeravanje

Ako je korisniku ili računaru omogućeno "Ograničeno preusmeravanje", moći će **preuzeti identitet bilo kog korisnika da bi pristupio nekim uslugama na računaru**.\
Zatim, ako **kompromitujete hash** ovog korisnika/računara, moći ćete **preuzeti identitet bilo kog korisnika** (čak i domenskih administratora) da bi pristupili nekim uslugama.

{% content-ref url="constrained-delegation.md" %}
[constrained-delegation.md](constrained-delegation.md)
{% endcontent-ref %}

### Ograničeno preusmeravanje zasnovano na resursima

Imajući **WRITE** privilegiju na objektu Active Directory-ja udaljenog računara omogućava izvršavanje koda sa **povišenim privilegijama**:

{% content-ref url="resource-based-constrained-delegation.md" %}
[resource-based-constrained-delegation.md](resource-based-constrained-delegation.md)
{% endcontent-ref %}

### Zloupotreba ACL-a

Kompromitovani korisnik može imati neke **interesantne privilegije nad nekim objektima domena** koje vam mogu omogućiti **lateralno kretanje**/**povišenje privilegija**.

{% content-ref url="acl-persistence-abuse/" %}
[acl-persistence-abuse](acl-persistence-abuse/)
{% endcontent-ref %}

### Zloupotreba usluge štampača

Otkrivanje **Službe štampača koja osluškuje** unutar domena može biti **zloupotrebljeno** za **dobijanje novih akreditacija** i **povišenje privilegija**.

{% content-ref url="acl-persistence-abuse/" %}
[acl-persistence-abuse](acl-persistence-abuse/)
{% endcontent-ref %}

### Zloupotreba sesija trećih strana

Ako **drugi korisnici pristupaju** **kompromitovanom** računaru, moguće je **prikupiti akreditacije iz memorije** i čak **ubaciti beacons u njihove procese** da bi se predstavljali kao oni.\
Obično će korisnici pristupiti sistemu putem RDP-a, pa evo kako izvesti nekoliko napada na sesije trećih strana putem RDP-a:

{% content-ref url="rdp-sessions-abuse.md" %}
[rdp-sessions-abuse.md](rdp-sessions-abuse.md)
{% endcontent-ref %}

### LAPS

**LAPS** pruža sistem za upravljanje **lozinkom lokalnog administratora** na računarima pridruženim domenu, obezbeđujući da je **slučajna**, jedinstvena i često **promenjena**. Ove lozinke se čuvaju u Active Directory-ju, a pristup se kontroliše putem ACL-a samo za ovlašćene korisnike. Sa dovoljnim dozvolama za pristup ovim lozinkama, moguće je preći na druge računare.

{% content-ref url="laps.md" %}
[laps.md](laps.md)
{% endcontent-ref %}

### Krađa sertifikata

**Prikupljanje sertifikata** sa kompromitovanog računara može biti način za povišenje privilegija unutar okruženja:

{% content-ref url="ad-certificates/certificate-theft.md" %}
[certificate-theft.md](ad-certificates/certificate-theft.md)
{% endcontent-ref %}

### Zloupotreba šablona sertifikata

Ako su konfigurisani **ranjivi šabloni**, moguće ih je zloupotrebiti za povišenje privilegija:

{% content-ref url="ad-certificates/domain-escalation.md" %}
[domain-escalation.md](ad-certificates/domain-escalation.md)
{% endcontent-ref %}

## Post-eksploatacija sa nalogom visokih privilegija

### Izvlačenje domenskih akreditacija

Kada dobijete privilegije **Administratora domena** ili čak bolje **Enterprise Admina**, možete **izvući** bazu podataka domena: _ntds.dit_.

[**Više informacija o DCSync napadu možete pronaći ovde**](dcsync.md).

[**Više informacija o tome kako ukrasti NTDS.dit možete pronaći ovde**](https://github.com/carlospolop/hacktricks/blob/rs/windows-hardening/active-directory-methodology/broken-reference/README.md)

### Povišenje privilegija kao trajna infekcija

Neke od tehnika koje su prethodno razmatrane mogu se koristiti za trajnu infekciju.\
Na primer, možete:

* Učiniti korisnike ranjivim na [**Kerberoast**](kerberoast.md)

```powershell
Set-DomainObject -Identity <korisničko_ime> -Set @{serviceprincipalname="fake/NOTHING"}r
```

* Učiniti korisnike ranjivim na [**ASREPRoast**](asreproast.md)

```powershell
Set-DomainObject -Identity <korisničko_ime> -XOR @{UserAccountControl=4194304}
```

* Dodeliti privilegije [**DCSync**](./#dcsync) korisniku

```powershell
Add-DomainObjectAcl -TargetIdentity "DC=SUB,DC=DOMAIN,DC=LOCAL" -PrincipalIdentity bfarmer -Rights DCSync
```

### Silver Ticket

Napad **Silver Ticket** kreira **legitimnu uslugu izdavanja tiketa (TGS) karticu** za određenu uslugu koristeći **NTLM hash** (na primer, hash PC naloga). Ova metoda se koristi za **pristup privilegijama usluge**.

{% content-ref url="silver-ticket.md" %}
[silver-ticket.md](silver-ticket.md)
{% endcontent-ref %}

### Golden Ticket

Napad **Golden Ticket** podrazumeva da napadač dobije pristup **NTLM hash-u krbtgt naloga** u okruženju Active Directory (AD). Ovaj nalog je poseban jer se koristi za potpisivanje svih **Ticket Granting Tiketa (TGT)**, koji su neophodni za autentifikaciju unutar AD mreže.

Kada napadač dobije ovaj hash, može kreirati **TGT-ove** za bilo koji nalog koji odabere (napad Silver ticket).

{% content-ref url="golden-ticket.md" %}
[golden-ticket.md](golden-ticket.md)
{% endcontent-ref %}

### Diamond Ticket

Ovo su kao zlatni tiketi koji su falsifikovani na način koji \*\*zaobilazi uobičajene

### **Persistencija domena putem sertifikata**

**Korišćenjem sertifikata takođe je moguće ostvariti perzistenciju sa visokim privilegijama unutar domena:**

{% content-ref url="ad-certificates/domain-persistence.md" %}
[domain-persistence.md](ad-certificates/domain-persistence.md)
{% endcontent-ref %}

### Grupa AdminSDHolder

Objekat **AdminSDHolder** u Active Directory-u obezbeđuje sigurnost **privilegovanih grupa** (kao što su Domain Admins i Enterprise Admins) primenom standardne **Access Control List (ACL)** na ove grupe kako bi se sprečile neovlaštene promene. Međutim, ova funkcionalnost može biti iskorišćena; ako napadač izmeni ACL AdminSDHolder-a kako bi dao pun pristup običnom korisniku, taj korisnik dobija široku kontrolu nad svim privilegovanim grupama. Ova sigurnosna mera, koja je namenjena zaštiti, može se vratiti kao bumerang, omogućavajući neovlašćeni pristup osim ako se pažljivo prati.

[**Više informacija o grupi AdminSDHolder ovde.**](privileged-groups-and-token-privileges.md#adminsdholder-group)

### DSRM akreditivi

Unutar svakog **Domain Controller (DC)**-a postoji **lokalni administratorski** nalog. Dobijanjem administratorskih prava na takvom računaru, lokalni Administrator hash može se izvući koristeći **mimikatz**. Nakon toga, potrebna je izmena registra da bi se **omogućila upotreba ovog lozinke**, što omogućava daljinski pristup lokalnom administratorskom nalogu.

{% content-ref url="dsrm-credentials.md" %}
[dsrm-credentials.md](dsrm-credentials.md)
{% endcontent-ref %}

### Perzistencija ACL-a

Možete **dodeliti** neke **posebne dozvole** korisniku nad određenim objektima domena koje će omogućiti korisniku **povećanje privilegija u budućnosti**.

{% content-ref url="acl-persistence-abuse/" %}
[acl-persistence-abuse](acl-persistence-abuse/)
{% endcontent-ref %}

### Sigurnosni deskriptori

**Sigurnosni deskriptori** se koriste za **čuvanje** **dozvola** koje **objekat** ima **nad** drugim **objektom**. Ako možete samo **napraviti** malu **promenu** u sigurnosnom deskriptoru objekta, možete dobiti veoma interesantne privilegije nad tim objektom, bez potrebe da budete član privilegovane grupe.

{% content-ref url="security-descriptors.md" %}
[security-descriptors.md](security-descriptors.md)
{% endcontent-ref %}

### Skeleton Key

Izmenite **LSASS** u memoriji da biste uspostavili **univerzalnu lozinku**, koja omogućava pristup svim korisničkim nalozima domena.

{% content-ref url="skeleton-key.md" %}
[skeleton-key.md](skeleton-key.md)
{% endcontent-ref %}

### Prilagođeni SSP

[Saznajte šta je SSP (Security Support Provider) ovde.](../authentication-credentials-uac-and-efs/#security-support-provider-interface-sspi)\
Možete kreirati **sopstveni SSP** da biste **uhvatili** u **čistom tekstu** **akreditive** koji se koriste za pristup mašini.\\

{% content-ref url="custom-ssp.md" %}
[custom-ssp.md](custom-ssp.md)
{% endcontent-ref %}

### DCShadow

Registruje **novi Domain Controller** u AD i koristi ga da **doda atribute** (SIDHistory, SPN...) na određene objekte **bez** ostavljanja **logova** o **izmenama**. Potrebne su privilegije DA i trebate biti unutar **root domena**.\
Imajte na umu da će se pojaviti ružni logovi ako koristite netačne podatke.

{% content-ref url="dcshadow.md" %}
[dcshadow.md](dcshadow.md)
{% endcontent-ref %}

### LAPS perzistencija

Ranije smo razgovarali o tome kako povećati privilegije ako imate **dovoljno dozvola za čitanje LAPS lozinki**. Međutim, ove lozinke se takođe mogu koristiti za **održavanje perzistencije**.\
Proverite:

{% content-ref url="laps.md" %}
[laps.md](laps.md)
{% endcontent-ref %}

## Eskalacija privilegija u šumi - Poverenje domena

Microsoft posmatra **šumu** kao sigurnosnu granicu. To znači da **kompromitovanje jednog domena može potencijalno dovesti do kompromitovanja cele šume**.

### Osnovne informacije

[**Poverenje domena**](http://technet.microsoft.com/en-us/library/cc759554\(v=ws.10\).aspx) je sigurnosni mehanizam koji omogućava korisniku iz jednog **domena** pristup resursima u drugom **domenu**. Ono stvara vezu između sistema za autentifikaciju dva domena, omogućavajući da se autentifikacija odvija bez problema. Kada domeni uspostave poverenje, razmenjuju i zadržavaju određene **ključeve** unutar svojih **Domain Controller (DC)**-a, koji su ključni za integritet poverenja.

U tipičnom scenariju, ako korisnik želi da pristupi usluzi u **poverenom domenu**, prvo mora zatražiti poseban tiket poznat kao **inter-realm TGT** od svog DC-a u sopstvenom domenu. Ovaj TGT je šifrovan sa deljenim **ključem** na koji su se oba domena saglasila. Korisnik zatim predstavlja ovaj TGT **DC-u poverenog domena** da bi dobio tiket za uslugu (**TGS**). Nakon uspešne validacije inter-realm TGT-a od strane DC-a poverenog domena, izdaje se TGS koji korisniku omogućava pristup usluzi.

**Koraci**:

1. **Klijentski računar** u **Domen 1** pokreće proces koristeći svoj **NTLM hash** da zatraži **Ticket Granting Ticket (TGT)** od svog **Domain Controller-a (DC1)**.
2. DC1 izdaje novi TGT ako je klijent uspešno autentifikovan.
3. Klijent zatim zahteva **inter-realm TGT** od DC1, koji je potreban za pristup resursima u **Domen 2**.
4. Inter-realm TGT je šifrovan sa **poverenim ključem** koji dele DC1 i DC2 kao deo dvosmernog poverenja između domena.
5. Klijent odnosi inter-realm TGT na **Domain Controller (DC2) Domena 2**.
6. DC2 proverava inter-realm TGT koristeći deljeni povereni ključ i, ako je validan, izdaje **Ticket Granting Service (TGS)** za server u Domen 2 kojem klijent želi da pristupi.
7. Na kraju, klijent predstavlja ovaj TGS serveru, koji je šifrovan sa hashom naloga servera, kako bi dobio pristup usluzi u Domen 2.

### Različita poverenja

Važno je primetiti da **poverenje može biti jednosmerno ili dvosmerno**. U dvosmernim opcijama, oba domena će međusobno verovati, ali u slučaju **jednosmernog** poverenja jedan od domena će biti **povereni** domen, a drugi **poverljivi** domen. U poslednjem slučaju, **samo ćete moći pristupiti resursima unutar poverljivog domena iz poverenog domena**.

Ako Domen A ver

#### Ostale razlike u **poverljivim odnosima**

* Poverljivi odnos može biti i **tranzitivan** (A veruje B, B veruje C, onda A veruje C) ili **netranzitivan**.
* Poverljivi odnos može biti postavljen kao **dvosmerno poverenje** (oba veruju jedno drugom) ili kao **jednosmerno poverenje** (samo jedan od njih veruje drugom).

### Put napada

1. **Nabrajanje** poverljivih odnosa
2. Proveriti da li neki **bezbednosni princip** (korisnik/grupa/računar) ima **pristup** resursima **druge domene**, možda putem unosa ACE ili putem pripadanja grupama druge domene. Potražite **odnose između domena** (verovatno je poverenje uspostavljeno iz tog razloga).
3. U ovom slučaju, kerberoast bi mogao biti još jedna opcija.
4. **Kompromitovati** naloge koji mogu **preći** između domena.

Napadači mogu pristupiti resursima u drugoj domeni putem tri osnovna mehanizma:

* **Članstvo u lokalnoj grupi**: Principali mogu biti dodati u lokalne grupe na računarima, kao što je grupa "Administratori" na serveru, što im omogućava značajnu kontrolu nad tim računarom.
* **Članstvo u grupi strane domene**: Principali takođe mogu biti članovi grupa unutar strane domene. Međutim, efikasnost ovog metoda zavisi od prirode poverenja i opsega grupe.
* **Kontrolne liste pristupa (ACL)**: Principali mogu biti navedeni u ACL-u, posebno kao entiteti u ACE-ovima unutar DACL-a, što im omogućava pristup određenim resursima. Za one koji žele dublje da se upuste u mehaniku ACL-a, DACL-a i ACE-ova, bela knjiga pod nazivom "[An ACE Up The Sleeve](https://specterops.io/assets/resources/an\_ace\_up\_the\_sleeve.pdf)" je neprocenjiv resurs.

### Eskalacija privilegija od deteta do roditelja u šumi

```
Get-DomainTrust

SourceName      : sub.domain.local    --> current domain
TargetName      : domain.local        --> foreign domain
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST       --> WITHIN_FOREST: Both in the same forest
TrustDirection  : Bidirectional       --> Trust direction (2ways in this case)
WhenCreated     : 2/19/2021 1:28:00 PM
WhenChanged     : 2/19/2021 1:28:00 PM
```

{% hint style="warning" %}
Postoje **2 pouzdane ključeve**, jedan za _Dete --> Roditelj_ i drugi za _Roditelj_ --> _Dete_.\
Možete pronaći onaj koji se koristi od strane trenutne domene pomoću:

```bash
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.my.domain.local
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\mcorp$"'
```
{% endhint %}

#### SID-History Injection

Eskalirajte kao Enterprise admin do deteta/roditeljske domene zloupotrebom poverenja sa SID-History ubrizgavanjem:

{% content-ref url="sid-history-injection.md" %}
[sid-history-injection.md](sid-history-injection.md)
{% endcontent-ref %}

#### Iskoristite konfiguraciju NC koja se može pisati

Razumevanje kako se može iskoristiti Configuration Naming Context (NC) je ključno. Configuration NC služi kao centralni repozitorijum za konfiguracione podatke u okviru šume u Active Directory (AD) okruženjima. Ovi podaci se replikuju na svaki Domain Controller (DC) unutar šume, pri čemu DC-ovi koji mogu da se pišu održavaju kopiju Configuration NC koja se može pisati. Da biste iskoristili ovo, morate imati **SYSTEM privilegije na DC-u**, po mogućstvu na DC-u deteta.

**Povežite GPO sa korenskim DC sajtom**

Container Sites Configuration NC-a sadrži informacije o svim sajtovima računara koji su pridruženi domeni unutar AD šume. Koristeći SYSTEM privilegije na bilo kom DC-u, napadači mogu povezati GPO-ove sa korenskim DC sajtovima. Ova akcija potencijalno kompromituje korensku domenu manipulacijom politika koje se primenjuju na ove sajtove.

Za detaljnije informacije, možete istražiti istraživanje o [Bypassing SID Filtering](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-4-bypass-sid-filtering-research).

**Kompromitujte bilo koji gMSA u šumi**

Vektor napada uključuje ciljanje privilegovanih gMSA unutar domene. KDS Root ključ, koji je neophodan za izračunavanje lozinki gMSA, se čuva unutar Configuration NC-a. Sa SYSTEM privilegijama na bilo kom DC-u, moguće je pristupiti KDS Root ključu i izračunati lozinke za bilo koji gMSA u šumi.

Detaljna analiza se može pronaći u diskusiji o [Golden gMSA Trust Attacks](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-5-golden-gmsa-trust-attack-from-child-to-parent).

**Napad promenom šeme**

Ova metoda zahteva strpljenje, čekanje na kreiranje novih privilegovanih AD objekata. Sa SYSTEM privilegijama, napadač može izmeniti AD šemu kako bi dao bilo kom korisniku potpunu kontrolu nad svim klasama. Ovo može dovesti do neovlašćenog pristupa i kontrole nad novokreiranim AD objektima.

Više informacija možete pronaći u [Schema Change Trust Attacks](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-6-schema-change-trust-attack-from-child-to-parent).

**Od DA do EA sa ADCS ESC5**

ADCS ESC5 ranjivost cilja kontrolu nad objektima javnog ključa infrastrukture (PKI) kako bi se kreirao šablon sertifikata koji omogućava autentifikaciju kao bilo koji korisnik u šumi. Pošto se PKI objekti nalaze u Configuration NC-u, kompromitacija DC-a deteta koji se može pisati omogućava izvršavanje ESC5 napada.

Više detalja o ovome možete pročitati u [From DA to EA with ESC5](https://posts.specterops.io/from-da-to-ea-with-esc5-f9f045aa105c). U scenarijima u kojima nema ADCS-a, napadač ima mogućnost da postavi neophodne komponente, kao što je opisano u [Escalating from Child Domain Admins to Enterprise Admins](https://www.pkisolutions.com/escalating-from-child-domains-admins-to-enterprise-admins-in-5-minutes-by-abusing-ad-cs-a-follow-up/).

### Spoljni šumski domen - Jednosmerni (ulazni) ili dvosmerni

```powershell
Get-DomainTrust
SourceName      : a.domain.local   --> Current domain
TargetName      : domain.external  --> Destination domain
TrustType       : WINDOWS-ACTIVE_DIRECTORY
TrustAttributes :
TrustDirection  : Inbound          --> Inboud trust
WhenCreated     : 2/19/2021 10:50:56 PM
WhenChanged     : 2/19/2021 10:50:56 PM
```

U ovom scenariju **vaš domen je poveren** od strane spoljnog domena, što vam daje **nepoznate dozvole** nad njim. Morate pronaći **koji principali vašeg domena imaju pristup spoljnom domenu** i zatim pokušati iskoristiti to:

{% content-ref url="external-forest-domain-oneway-inbound.md" %}
[external-forest-domain-oneway-inbound.md](external-forest-domain-oneway-inbound.md)
{% endcontent-ref %}

### Spoljni šumski domen - Jednosmerno (izlazno)

```powershell
Get-DomainTrust -Domain current.local

SourceName      : current.local   --> Current domain
TargetName      : external.local  --> Destination domain
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Outbound        --> Outbound trust
WhenCreated     : 2/19/2021 10:15:24 PM
WhenChanged     : 2/19/2021 10:15:24 PM
```

U ovom scenariju, **vaš domen** poverava neke **privilegije** principalu iz **drugih domena**.

Međutim, kada je **domein poveren** od strane poverljivog domena, povereni domen **kreira korisnika** sa **predvidljivim imenom** koji koristi kao **lozinku poverljivu lozinku**. To znači da je moguće **pristupiti korisniku iz poverljivog domena da bi se ušlo u povereni domen** kako bi se izvršila enumeracija i pokušalo da se dobiju više privilegija:

{% content-ref url="external-forest-domain-one-way-outbound.md" %}
[external-forest-domain-one-way-outbound.md](external-forest-domain-one-way-outbound.md)
{% endcontent-ref %}

Još jedan način da se kompromituje povereni domen je pronalaženje [**SQL poverenog linka**](abusing-ad-mssql.md#mssql-trusted-links) koji je kreiran u **suprotnom smeru** od domenskog poverenja (što nije vrlo uobičajeno).

Još jedan način da se kompromituje povereni domen je čekanje na mašini na kojoj **korisnik iz poverenog domena može pristupiti** kako bi se prijavio putem **RDP-a**. Zatim, napadač bi mogao ubaciti kod u proces RDP sesije i **pristupiti domenu porekla žrtve** odatle.\
Osim toga, ako je **žrtva montirala svoj hard disk**, iz procesa RDP sesije napadač bi mogao smeštati **bekdore** u **startap folder hard diska**. Ova tehnika se naziva **RDPInception**.

{% content-ref url="rdp-sessions-abuse.md" %}
[rdp-sessions-abuse.md](rdp-sessions-abuse.md)
{% endcontent-ref %}

### Mitigacija zloupotrebe poverenja domena

### **SID filtriranje:**

* Rizik od napada koji koriste atribut SID istorije preko šumskih poverenja umanjuje SID filtriranje, koje je podrazumevano aktivirano na svim međušumskim poverenjima. Ovo se zasniva na pretpostavci da su unutaršumska poverenja sigurna, uzimajući u obzir šumu, a ne domen, kao granicu bezbednosti prema stavu Microsoft-a.
* Međutim, postoji kvaka: SID filtriranje može poremetiti aplikacije i korisnički pristup, što dovodi do povremenog deaktiviranja.

### **Selektivna autentifikacija:**

* Za međušumska poverenja, primena selektivne autentifikacije osigurava da se korisnici iz dve šume ne autentifikuju automatski. Umesto toga, potrebne su eksplicitne dozvole korisnicima da pristupe domenima i serverima unutar poverljivog domena ili šume.
* Važno je napomenuti da ove mere ne štite od iskorišćavanja upisivog Configuration Naming Context (NC) ili napada na nalog za poverenje.

[**Više informacija o poverenju domena na ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain)

## AD -> Azure & Azure -> AD

{% embed url="https://cloud.hacktricks.xyz/pentesting-cloud/azure-security/az-lateral-movements/azure-ad-connect-hybrid-identity" %}

## Neke opšte odbrane

[**Saznajte više o zaštiti akreditiva ovde.**](../stealing-credentials/credentials-protections.md)\\

### **Odbrambene mere za zaštitu akreditiva**

* **Ograničenja domenskih administratora**: Preporučuje se da domenski administratori mogu se prijaviti samo na kontrolere domena, izbegavajući njihovu upotrebu na drugim hostovima.
* **Privilegije servisnih naloga**: Servisi ne bi trebali da se pokreću sa privilegijama domenskog administratora (DA) kako bi se održala bezbednost.
* **Vremensko ograničenje privilegija**: Za zadatke koji zahtevaju privilegije domenskog administratora, njihovo trajanje treba ograničiti. To se može postići pomoću: `Add-ADGroupMember -Identity ‘Domain Admins’ -Members newDA -MemberTimeToLive (New-TimeSpan -Minutes 20)`

### **Implementacija tehnika obmane**

* Implementacija obmane podrazumeva postavljanje zamki, poput lažnih korisnika ili računara, sa funkcijama kao što su lozinke koje ne ističu ili su označene kao pouzdane za delegaciju. Detaljan pristup uključuje kreiranje korisnika sa određenim pravima ili njihovo dodavanje u grupe visokih privilegija.
* Praktičan primer uključuje korišćenje alata kao što je: `Create-DecoyUser -UserFirstName user -UserLastName manager-uncommon -Password Pass@123 | DeployUserDeception -UserFlag PasswordNeverExpires -GUID d07da11f-8a3d-42b6-b0aa-76c962be719a -Verbose`
* Više o implementaciji tehnika obmane može se pronaći na [Deploy-Deception na GitHub-u](https://github.com/samratashok/Deploy-Deception).

### **Identifikacija obmane**

* **Za korisničke objekte**: Sumnjivi pokazatelji uključuju atipičan ObjectSID, retke prijave, datume kreiranja i nizak broj loših lozinki.
* **Opšti pokazatelji**: Upoređivanje atributa potencijalnih lažnih objekata sa atributima stvarnih objekata može otkriti neusaglašenosti. Alati poput [HoneypotBuster](https://github.com/JavelinNetworks/HoneypotBuster) mogu pomoći u identifikaciji takvih obmana.

### **Bypassing sistema za detekciju**

* **Bypass detekcije Microsoft ATA**:
* **Enumeracija korisnika**: Izbeći enumeraciju sesija na kontrolerima domena kako bi se sprečila detekcija ATA.
* **Impersonacija tiketa**: Korišćenje **aes** ključeva za kreiranje tiketa pomaže izbegavanju detekcije tako što se ne vrši degradacija na NTLM.
* **DCSync napadi**: Preporučuje se izvršavanje sa ne-Domain Controllera kako bi se izbegla detekcija ATA, jer direktno izvršavanje sa Domain Controllera će izazvati upozorenja.

## Reference

* [http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/](http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/)
* [https://www.labofapenetrationtester.com/2018/10/deploy-deception.html](https://www.labofapenetrationtester.com/2018/10/deploy-deception.html)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
