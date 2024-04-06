# AD Certificates

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Uvod

### Komponente sertifikata

* **Subject** sertifikata označava njegovog vlasnika.
* **Javni ključ** je uparen sa privatnim ključem kako bi se sertifikat povezao sa svojim pravim vlasnikom.
* **Period važenja**, definisan datumima **NotBefore** i **NotAfter**, označava efektivno trajanje sertifikata.
* Jedinstveni **Serijski broj**, koji obezbeđuje Sertifikacioni Autoritet (CA), identifikuje svaki sertifikat.
* **Izdavalac** se odnosi na CA koji je izdao sertifikat.
* **SubjectAlternativeName** omogućava dodatna imena za subjekta, poboljšavajući fleksibilnost identifikacije.
* **Osnovna ograničenja** identifikuju da li je sertifikat za CA ili krajnji entitet i definišu ograničenja upotrebe.
* **Proširene namene ključeva (EKU)** razgraničavaju specifične svrhe sertifikata, poput potpisivanja koda ili enkripcije e-pošte, putem objektnih identifikatora (OID).
* **Algoritam potpisa** specificira metodu za potpisivanje sertifikata.
* **Potpis**, kreiran sa privatnim ključem izdavaoca, garantuje autentičnost sertifikata.

### Posebne razmatranja

* **Subject Alternative Names (SANs)** proširuju primenljivost sertifikata na više identiteta, što je ključno za servere sa više domena. Bezbedni procesi izdavanja su od vitalnog značaja kako bi se izbegli rizici od impersonacije od strane napadača koji manipulišu specifikacijom SAN-a.

### Sertifikacioni Autoriteti (CA) u Active Directory (AD)

AD CS priznaje CA sertifikate u AD šumi putem određenih kontejnera, pri čemu svaki ima jedinstvene uloge:

* Kontejner **Certification Authorities** sadrži sertifikate poverenih korenskih CA.
* Kontejner **Enrolment Services** detaljiše Enterprise CA i njihove šablone sertifikata.
* Objekat **NTAuthCertificates** uključuje CA sertifikate ovlašćene za AD autentifikaciju.
* Kontejner **AIA (Authority Information Access)** olakšava validaciju lanca sertifikata sa posrednim i prekograničnim CA sertifikatima.

### Sticanje sertifikata: Tok zahteva za klijentski sertifikat

1. Proces zahteva počinje tako što klijenti pronalaze Enterprise CA.
2. Nakon generisanja para javnog-privatnog ključa, kreira se CSR koji sadrži javni ključ i druge detalje.
3. CA procenjuje CSR u odnosu na dostupne šablone sertifikata, izdajući sertifikat na osnovu dozvola šablona.
4. Nakon odobrenja, CA potpisuje sertifikat svojim privatnim ključem i vraća ga klijentu.

### Šabloni sertifikata

Definisani unutar AD, ovi šabloni opisuju podešavanja i dozvole za izdavanje sertifikata, uključujući dozvoljene EKU i prava za upisivanje ili izmenu, što je ključno za upravljanje pristupom sertifikacionim uslugama.

## Upisivanje sertifikata

Proces upisivanja sertifikata pokreće administrator koji **kreira šablon sertifikata**, koji zatim **objavljuje** Enterprise Certificate Authority (CA). To čini šablon dostupnim za upisivanje klijenta, korak koji se postiže dodavanjem imena šablona u polje `certificatetemplates` objekta Active Directory.

Da bi klijent zatražio sertifikat, moraju mu biti dodeljena **prava upisivanja**. Ova prava se definišu putem bezbednosnih deskriptora na šablonu sertifikata i samom Enterprise CA. Dozvole moraju biti dodeljene na oba mesta da bi zahtev bio uspešan.

### Prava upisivanja šablona

Ova prava se specificiraju putem unosa za kontrolu pristupa (ACE), koji detaljišu dozvole poput:

* **Certificate-Enrollment** i **Certificate-AutoEnrollment** prava, svako povezano sa specifičnim GUID-om.
* **ExtendedRights**, omogućavajući sve proširene dozvole.
* **FullControl/GenericAll**, pružajući potpunu kontrolu nad šablonom.

### Prava upisivanja Enterprise CA

Prava CA su definisana u njegovom bezbednosnom deskriptoru, koji je dostupan putem konzole za upravljanje Certificate Authority. Neke postavke čak omogućavaju korisnicima sa niskim privilegijama daljinski pristup, što može predstavljati bezbednosni rizik.

### Dodatne kontrole izdavanja

Mogu se primeniti određene kontrole, kao što su:

* **Odobrenje menadžera**: Stavlja zahteve u stanje čekanja dok ih ne odobri menadžer sertifikata.
* **Enrolment Agents i Authorized Signatures**: Određuju broj potrebnih potpisa na CSR-u i neophodne Application Policy OIDs.

### Metode zahteva za sertifikate

Sertifikati se mogu zahtevati putem:

1. **Windows Client Certificate Enrollment Protocol** (MS-WCCE), koristeći DCOM interfejse.
2. **ICertPassage Remote Protocol** (MS-ICPR), putem imenovanih cevi ili TCP/IP-a.
3. **Veb interfejs za upisivanje sertifikata**, sa instaliranom ulogom Certificate Authority Web Enrollment.
4. **Certificate Enrollment Service** (CES), u kombinaciji sa Certificate Enrollment Policy (CEP) servisom.
5. **Network Device Enrollment Service** (NDES) za mrežne uređaje, koristeći Simple Certificate Enrollment Protocol (SCEP).

Windows korisnici takođe mogu zahtevati sertifikate putem grafičkog interfejsa (`certmgr.msc` ili `certlm.msc`) ili alata komandne linije (`certreq.exe` ili PowerShell-ov `Get-Certificate` komanda).

```powershell
# Example of requesting a certificate using PowerShell
Get-Certificate -Template "User" -CertStoreLocation "cert:\\CurrentUser\\My"
```

## Autentifikacija sertifikatom

Active Directory (AD) podržava autentifikaciju putem sertifikata, uglavnom koristeći protokole **Kerberos** i **Secure Channel (Schannel)**.

### Proces autentifikacije Kerberos

U procesu autentifikacije Kerberos, zahtev korisnika za Ticket Granting Ticket (TGT) se potpisuje korišćenjem **privatnog ključa** korisnikovog sertifikata. Ovaj zahtev prolazi kroz nekoliko validacija od strane kontrolera domena, uključujući **validnost**, **putanju** i **status opoziva** sertifikata. Validacije takođe uključuju proveru da li sertifikat potiče od pouzdanog izvora i potvrdu prisustva izdavaoca u **NTAUTH skladištu sertifikata**. Uspesne validacije rezultiraju izdavanjem TGT-a. **`NTAuthCertificates`** objekat u AD-u, koji se nalazi na:

```bash
CN=NTAuthCertificates,CN=Public Key Services,CN=Services,CN=Configuration,DC=<domain>,DC=<com>
```

je ključno za uspostavljanje poverenja za autentifikaciju sertifikata.

### Autentifikacija sigurnog kanala (Schannel)

Schannel omogućava sigurne TLS/SSL veze, gde tokom rukovanja, klijent predstavlja sertifikat koji, ako se uspešno validira, autorizuje pristup. Mapiranje sertifikata na AD nalog može uključivati funkciju **S4U2Self** Kerberosa ili **Subject Alternative Name (SAN)** sertifikata, među ostalim metodama.

### Nabrojavanje AD sertifikatnih servisa

AD-ovi sertifikatni servisi mogu biti nabrojani putem LDAP upita, otkrivajući informacije o **Enterprise Certificate Authorities (CA)** i njihovim konfiguracijama. Ovo je dostupno svakom korisniku autentifikovanom u domenu bez posebnih privilegija. Alati poput [**Certify**](https://github.com/GhostPack/Certify) i [**Certipy**](https://github.com/ly4k/Certipy) se koriste za nabrojavanje i procenu ranjivosti u AD CS okruženjima.

Komande za korišćenje ovih alata uključuju:

```bash
# Enumerate trusted root CA certificates and Enterprise CAs with Certify
Certify.exe cas
# Identify vulnerable certificate templates with Certify
Certify.exe find /vulnerable

# Use Certipy for enumeration and identifying vulnerable templates
certipy find -vulnerable -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128

# Enumerate Enterprise CAs and certificate templates with certutil
certutil.exe -TCAInfo
certutil -v -dstemplate
```

## Reference

* [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)
* [https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html](https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
