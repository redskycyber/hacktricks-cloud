# AD CS Domain Persistence

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Ovo je sažetak tehnika postojanosti domena koje su podeljene u** [**https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf**](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf). Proverite ga za dalje detalje.

## Forgeovanje sertifikata sa ukradenim CA sertifikatima - DPERSIST1

Kako možete da utvrdite da li je sertifikat CA sertifikat?

Može se utvrditi da je sertifikat CA sertifikat ako se ispunjavaju nekoliko uslova:

* Sertifikat je smešten na CA serveru, sa svojim privatnim ključem obezbeđenim DPAPI mašine, ili hardverom kao što je TPM/HSM ako operativni sistem to podržava.
* Polja Izdavača i Subjekta sertifikata se podudaraju sa razlikovanim imenom CA.
* "CA verzija" ekstenzija je prisutna isključivo u CA sertifikatima.
* Sertifikat nema polja Proširene upotrebe ključa (EKU).

Da biste izvukli privatni ključ ovog sertifikata, podržana metoda putem ugrađenog grafičkog korisničkog interfejsa je alatka `certsrv.msc` na CA serveru. Međutim, ovaj sertifikat se ne razlikuje od ostalih smeštenih u sistemu; stoga se mogu primeniti metode poput [THEFT2 tehnike](certificate-theft.md#user-certificate-theft-via-dpapi-theft2) za izvlačenje.

Sertifikat i privatni ključ mogu se takođe dobiti korišćenjem Certipy sa sledećom komandom:

```bash
certipy ca 'corp.local/administrator@ca.corp.local' -hashes :123123.. -backup
```

Nakon što ste dobili CA sertifikat i njegov privatni ključ u `.pfx` formatu, alati poput [ForgeCert](https://github.com/GhostPack/ForgeCert) mogu se koristiti za generisanje validnih sertifikata:

```bash
# Generating a new certificate with ForgeCert
ForgeCert.exe --CaCertPath ca.pfx --CaCertPassword Password123! --Subject "CN=User" --SubjectAltName localadmin@theshire.local --NewCertPath localadmin.pfx --NewCertPassword Password123!

# Generating a new certificate with certipy
certipy forge -ca-pfx CORP-DC-CA.pfx -upn administrator@corp.local -subject 'CN=Administrator,CN=Users,DC=CORP,DC=LOCAL'

# Authenticating using the new certificate with Rubeus
Rubeus.exe asktgt /user:localdomain /certificate:C:\ForgeCert\localadmin.pfx /password:Password123!

# Authenticating using the new certificate with certipy
certipy auth -pfx administrator_forged.pfx -dc-ip 172.16.126.128
```

{% hint style="warning" %}
Korisnik koji je cilj za falsifikovanje sertifikata mora biti aktivan i sposoban da se autentifikuje u Active Directory-u da bi proces bio uspešan. Falsifikacija sertifikata za posebne naloge poput krbtgt je neefikasna.
{% endhint %}

Ovaj falsifikovani sertifikat će biti **važeći** do navedenog datuma isteka i **sve dok je sertifikat korenskog CA važeći** (obično od 5 do **10+ godina**). Takođe je važeći za **mašine**, pa uz pomoć **S4U2Self**, napadač može **održavati postojanost na bilo kojoj mašini u domenu** sve dok je sertifikat CA važeći.\
Osim toga, **generisani sertifikati** ovom metodom **ne mogu biti povučeni** jer CA nije svestan njihovog postojanja.

## Poverenje u lažne CA sertifikate - DPERSIST2

Objekat `NTAuthCertificates` je definisan da sadrži jedan ili više **CA sertifikata** unutar svog atributa `cacertificate`, koji koristi Active Directory (AD). Proces verifikacije od strane **kontrolera domena** uključuje proveru objekta `NTAuthCertificates` za unos koji se podudara sa **CA koji je naveden** u polju Izdavač autentičnog **sertifikata**. Autentifikacija se nastavlja ako se pronađe podudaranje.

Napadač može dodati samopotpisani CA sertifikat u objekat `NTAuthCertificates`, pod uslovom da ima kontrolu nad ovim AD objektom. Obično samo članovi grupe **Enterprise Admin**, zajedno sa **Domain Admins** ili **Administratorima** u **korenskom domenu šume**, imaju dozvolu da izmene ovaj objekat. Oni mogu izmeniti objekat `NTAuthCertificates` koristeći `certutil.exe` sa komandom `certutil.exe -dspublish -f C:\Temp\CERT.crt NTAuthCA126`, ili korišćenjem [**PKI Health Tool**](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/import-third-party-ca-to-enterprise-ntauth-store#method-1---import-a-certificate-by-using-the-pki-health-tool).

Ova mogućnost je posebno relevantna kada se koristi u kombinaciji sa prethodno opisanom metodom koja uključuje ForgeCert za dinamičko generisanje sertifikata.

## Zlonamerna konfiguracija - DPERSIST3

Mogućnosti za **postojanost** putem **modifikacija bezbednosnog opisa komponenti AD CS** su brojne. Modifikacije opisane u odeljku "[Domain Escalation](domain-escalation.md)" mogu biti zlonamerne implementirane od strane napadača sa povišenim pristupom. To uključuje dodavanje "kontrolnih prava" (npr. WriteOwner/WriteDACL/itd.) osetljivim komponentama kao što su:

* **Računar AD servera CA**
* **RPC/DCOM server AD servera CA**
* Bilo koji **potomak AD objekta ili kontejnera** u **`CN=Public Key Services,CN=Services,CN=Configuration,DC=<DOMAIN>,DC=<COM>`** (na primer, kontejner Certificate Templates, kontejner Certification Authorities, objekat NTAuthCertificates, itd.)
* **AD grupe kojima su podrazumevano delegirana prava za kontrolu AD CS** ili od strane organizacije (kao što je ugrađena grupa Cert Publishers i bilo koji od njenih članova)

Primer zlonamerne implementacije uključuje napadača koji ima **povišene dozvole** u domenu, dodavanje dozvole **`WriteOwner`** na podrazumevani **`User`** šablon sertifikata, pri čemu je napadač princip za tu dozvolu. Da bi iskoristio ovo, napadač bi prvo promenio vlasništvo šablona **`User`** na sebe. Nakon toga, **`mspki-certificate-name-flag`** bi bio postavljen na **1** na šablonu da bi se omogućilo **`ENROLLEE_SUPPLIES_SUBJECT`**, što omogućava korisniku da pruži alternativno ime subjekta u zahtevu. Nakon toga, napadač bi mogao da se **upiše** koristeći **šablon**, birajući ime **administratora domena** kao alternativno ime, i koristiti dobijeni sertifikat za autentifikaciju kao DA.
