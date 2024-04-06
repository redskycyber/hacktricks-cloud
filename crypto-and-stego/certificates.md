# Certificates

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da biste lako izgradili i **automatizovali radne tokove** pokretane najnaprednijim alatima zajednice na svetu.\
Dobijte pristup danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Šta je Sertifikat

**Sertifikat javnog ključa** je digitalni ID koji se koristi u kriptografiji da bi se dokazalo da neko poseduje javni ključ. Uključuje detalje ključa, identitet vlasnika (subjekta) i digitalni potpis od pouzdane autoritete (izdavaoca). Ako softver veruje izdavaocu i potpis je validan, sigurna komunikacija sa vlasnikom ključa je moguća.

Sertifikati se uglavnom izdaju od strane [autoriteta za sertifikaciju](https://en.wikipedia.org/wiki/Certificate\_authority) (CA) u postavci [infrastrukture javnog ključa](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI). Drugi metod je [mreža poverenja](https://en.wikipedia.org/wiki/Web\_of\_trust), gde korisnici direktno verifikuju ključeve jedni drugih. Uobičajeni format za sertifikate je [X.509](https://en.wikipedia.org/wiki/X.509), koji se može prilagoditi za specifične potrebe kako je opisano u RFC 5280.

## Zajednička Polja x509

### **Zajednička Polja u x509 Sertifikatima**

U x509 sertifikatima, nekoliko **polja** igraju ključne uloge u osiguravanju validnosti i sigurnosti sertifikata. Evo razbijanja ovih polja:

* **Broj Verzije** označava verziju formata x509.
* **Seriski Broj** jedinstveno identifikuje sertifikat unutar sistema Autoriteta za Sertifikaciju (CA), uglavnom za praćenje opoziva.
* Polje **Subjekat** predstavlja vlasnika sertifikata, koji može biti mašina, pojedinac ili organizacija. Uključuje detaljne identifikacije kao što su:
* **Uobičajeno Ime (CN)**: Domeni obuhvaćeni sertifikatom.
* **Država (C)**, **Lokalitet (L)**, **Država ili Pokrajina (ST, S, ili P)**, **Organizacija (O)** i **Organizaciona Jedinica (OU)** pružaju geografske i organizacione detalje.
* **Distingovano Ime (DN)** obuhvata punu identifikaciju subjekta.
* **Izdavaoc** detalji ko je verifikovao i potpisao sertifikat, uključujući slična podpolja kao Subjekat za CA.
* **Period Važenja** obeležen je vremenskim oznakama **Nije Pre** i **Nije Posle**, osiguravajući da sertifikat nije korišćen pre ili posle određenog datuma.
* Odeljak **Javnog Ključa**, ključan za sigurnost sertifikata, specificira algoritam, veličinu i druge tehničke detalje javnog ključa.
* **x509v3 proširenja** poboljšavaju funkcionalnost sertifikata, specificirajući **Upotrebu Ključa**, **Proširenu Upotrebu Ključa**, **Alternativno Ime Subjekta** i druge osobine za fino podešavanje primene sertifikata.

#### **Upotreba Ključa i Proširenja**

* **Upotreba Ključa** identifikuje kriptografske primene javnog ključa, poput digitalnog potpisa ili šifrovanja ključem.
* **Proširena Upotreba Ključa** dodatno sužava slučajeve upotrebe sertifikata, npr. za autentikaciju TLS servera.
* **Alternativno Ime Subjekta** i **Osnovno Ograničenje** definišu dodatna imena hostova obuhvaćena sertifikatom i da li je to CA ili sertifikat entiteta.
* Identifikatori poput **Identifikatora Ključa Subjekta** i **Identifikatora Ključa Autoriteta** osiguravaju jedinstvenost i mogućnost praćenja ključeva.
* **Pristup Informacijama o Autoritetu** i **Tačke Distribucije CRL-a** pružaju putanje za verifikaciju izdavaoca CA i proveru statusa opoziva sertifikata.
* **CT Pre-sertifikat SCT-ovi** nude transparentne logove, ključne za javno poverenje u sertifikat.

```python
# Example of accessing and using x509 certificate fields programmatically:
from cryptography import x509
from cryptography.hazmat.backends import default_backend

# Load an x509 certificate (assuming cert.pem is a certificate file)
with open("cert.pem", "rb") as file:
cert_data = file.read()
certificate = x509.load_pem_x509_certificate(cert_data, default_backend())

# Accessing fields
serial_number = certificate.serial_number
issuer = certificate.issuer
subject = certificate.subject
public_key = certificate.public_key()

print(f"Serial Number: {serial_number}")
print(f"Issuer: {issuer}")
print(f"Subject: {subject}")
print(f"Public Key: {public_key}")
```

### **Razlika između OCSP i CRL distributivnih tačaka**

**OCSP** (**RFC 2560**) uključuje klijenta i odgovarača koji zajedno proveravaju da li je digitalni javni ključ sertifikata povučen, bez potrebe za preuzimanjem punog **CRL**-a. Ovaj metod je efikasniji od tradicionalnog **CRL**-a, koji pruža listu povučenih serijskih brojeva sertifikata, ali zahteva preuzimanje potencijalno velike datoteke. CRL-ovi mogu sadržati do 512 unosa. Više detalja dostupno je [ovde](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm).

### **Šta je Transparentnost sertifikata**

Transparentnost sertifikata pomaže u borbi protiv pretnji povezanih sa sertifikatima osiguravajući da izdavanje i postojanje SSL sertifikata budu vidljivi vlasnicima domena, CA-ovima i korisnicima. Njeni ciljevi su:

* Sprječavanje CA-ova da izdaju SSL sertifikate za domen bez znanja vlasnika domena.
* Uspostavljanje otvorenog sistema revizije za praćenje greškom ili zlonamerno izdatih sertifikata.
* Zaštita korisnika od lažnih sertifikata.

#### **Sertifikatni zapisi**

Sertifikatni zapisi su javno proverljivi, samo za dodavanje zapisi sertifikata, održavani od strane mrežnih servisa. Ovi zapisi pružaju kriptografske dokaze u svrhe revizije. Izdavači i javnost mogu podnositi sertifikate ovim zapisima ili ih upitati za verifikaciju. Iako tačan broj serverskih zapisa nije fiksan, očekuje se da ih globalno bude manje od hiljadu. Ovi serveri mogu biti nezavisno upravljani od strane CA-ova, ISP-ova ili bilo koje zainteresovane entitete.

#### **Upit**

Za istraživanje sertifikatnih zapisa Transparentnosti sertifikata za bilo koji domen, posetite [https://crt.sh/](https://crt.sh).

Različiti formati postoje za skladištenje sertifikata, svaki sa svojim slučajevima upotrebe i kompatibilnošću. Ovaj sažetak obuhvata glavne formate i pruža smernice o konvertovanju između njih.

## **Formati**

### **PEM Format**

* Najčešće korišćen format za sertifikate.
* Zahteva odvojene datoteke za sertifikate i privatne ključeve, kodirane u Base64 ASCII.
* Česte ekstenzije: .cer, .crt, .pem, .key.
* Prvenstveno korišćen od strane Apache i sličnih servera.

### **DER Format**

* Binarni format sertifikata.
* Nedostaje "BEGIN/END CERTIFICATE" izjave koje se nalaze u PEM datotekama.
* Česte ekstenzije: .cer, .der.
* Često korišćen sa Java platformama.

### **P7B/PKCS#7 Format**

* Skladišten u Base64 ASCII, sa ekstenzijama .p7b ili .p7c.
* Sadrži samo sertifikate i lanac sertifikata, isključujući privatni ključ.
* Podržan od strane Microsoft Windows i Java Tomcat.

### **PFX/P12/PKCS#12 Format**

* Binarni format koji uključuje serverske sertifikate, posredne sertifikate i privatne ključeve u jednoj datoteci.
* Ekstenzije: .pfx, .p12.
* Glavno korišćen na Windows platformi za uvoz i izvoz sertifikata.

### **Konvertovanje formata**

**PEM konverzije** su esencijalne za kompatibilnost:

* **x509 u PEM**

```bash
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```

* **PEM u DER**

```bash
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```

* **DER u PEM**

```bash
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```

* **PEM u P7B**

```bash
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```

* **PKCS7 u PEM**

```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```

**PFX konverzije** su ključne za upravljanje sertifikatima na Windows operativnom sistemu:

* **PFX u PEM**

```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```

* **PFX u PKCS#8** uključuje dva koraka:

1. Konvertuj PFX u PEM

```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```

2. Konvertuj PEM u PKCS8

```bash
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```

* **P7B u PFX** takođe zahteva dve komande:

1. Konvertuj P7B u CER

```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```

2. Konvertujte CER i privatni ključ u PFX

```bash
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile cacert.cer
```

***

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da biste lako izgradili i **automatizovali radne tokove** pokretane najnaprednijim alatima zajednice na svetu.\
Dobijte pristup danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
