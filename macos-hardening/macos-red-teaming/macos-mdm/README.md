# macOS MDM

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Da biste saznali više o macOS MDM-ovima pogledajte:**

* [https://www.youtube.com/watch?v=ku8jZe-MHUU](https://www.youtube.com/watch?v=ku8jZe-MHUU)
* [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe)

## Osnove

### **Pregled MDM (Mobile Device Management)**

[Mobile Device Management](https://en.wikipedia.org/wiki/Mobile\_device\_management) (MDM) se koristi za upravljanje različitim uređajima krajnjih korisnika kao što su pametni telefoni, prenosni računari i tableti. Posebno za Apple platforme (iOS, macOS, tvOS), uključuje set specijalizovanih funkcija, API-ja i praksi. Funkcionisanje MDM-a zavisi od kompatibilnog MDM servera, koji može biti komercijalno dostupan ili open-source, i mora podržavati [MDM protokol](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Ključni elementi uključuju:

* Centralizovana kontrola nad uređajima.
* Zavisnost od MDM servera koji se pridržava MDM protokola.
* Mogućnost MDM servera da šalje različite komande uređajima, na primer, daljinsko brisanje podataka ili instalacija konfiguracije.

### **Osnove DEP (Device Enrollment Program)**

[Device Enrollment Program](https://www.apple.com/business/site/docs/DEP\_Guide.pdf) (DEP) koji nudi Apple olakšava integraciju Mobile Device Management (MDM) omogućavajući konfiguraciju bez dodira za iOS, macOS i tvOS uređaje. DEP automatizuje proces registracije, omogućavajući uređajima da budu operativni odmah po izlasku iz kutije, sa minimalnom intervencijom korisnika ili administratora. Ključni aspekti uključuju:

* Omogućava uređajima da se automatski registruju kod predefinisanog MDM servera prilikom prvog aktiviranja.
* Pretežno korisno za potpuno nove uređaje, ali se takođe može primeniti i na uređaje koji se ponovno konfigurišu.
* Olakšava jednostavnu instalaciju, čime se uređaji brzo pripremaju za organizacionu upotrebu.

### **Bezbednosno razmatranje**

Važno je napomenuti da, iako je olakšana registracija putem DEP-a korisna, može predstavljati i bezbednosne rizike. Ako se ne primenjuju adekvatne zaštitne mere prilikom registracije putem MDM-a, napadači mogu iskoristiti ovaj pojednostavljeni proces da registruju svoj uređaj na MDM serveru organizacije, predstavljajući se kao korporativni uređaj.

{% hint style="danger" %}
**Bezbednosno upozorenje**: Pojednostavljena registracija putem DEP-a može potencijalno omogućiti neovlaštenu registraciju uređaja na MDM serveru organizacije ako nisu preduzete odgovarajuće mere zaštite.
{% endhint %}

### Osnove Šta je SCEP (Simple Certificate Enrolment Protocol)?

* Relativno stari protokol, kreiran pre širokog prihvatanja TLS i HTTPS.
* Klijentima pruža standardizovan način slanja **Certificate Signing Request** (CSR) radi dobijanja sertifikata. Klijent će zatražiti od servera da mu izda potpisan sertifikat.

### Šta su konfiguracioni profili (poznati i kao mobileconfigs)?

* Zvaničan način Apple-a za **postavljanje/primenu sistemskih konfiguracija**.
* Format datoteke koji može sadržati više payloada.
* Zasnovan na property listama (XML vrsta).
* "mogu biti potpisani i šifrovani radi provere porekla, obezbeđivanja integriteta i zaštite sadržaja." Osnove - Strana 70, iOS Security Guide, januar 2018.

## Protokoli

### MDM

* Kombinacija APNs (**Apple servera**) + RESTful API-ja (**MDM** **vendor** serveri)
* **Komunikacija** se odvija između uređaja i servera povezanog sa proizvodom za **upravljanje uređajima**
* **Komande** se dostavljaju sa MDM servera uređaju u obliku **plist-kodiranih rečnika**
* Sve preko **HTTPS-a**. MDM serveri mogu biti (i obično jesu) pinovani.
* Apple dodeljuje MDM vendoru **APNs sertifikat** za autentifikaciju

### DEP

* **3 API-ja**: 1 za prodavce, 1 za MDM vendore, 1 za identitet uređaja (nedokumentovano):
* Takođe poznat kao [DEP "cloud service" API](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Koristi se od strane MDM servera za povezivanje DEP profila sa određenim uređajima.
* [DEP API koji koriste ovlašćeni prodavci Apple-a](https://applecareconnect.apple.com/api-docs/depuat/html/WSImpManual.html) za registraciju uređaja, proveru statusa registracije i proveru statusa transakcije.
* Nedokumentovani privatni DEP API. Koristi se od strane Apple uređaja za zahtevanje DEP profila. Na macOS-u, binarna datoteka `cloudconfigurationd` je odgovorna za komunikaciju preko ovog API-ja.
* Moderniji i zasnovan na **JSON**-u (za razliku od plist-a)
* Apple dodeljuje MDM vendoru **OAuth token**

**DEP "cloud service" API**

* RESTful
* sinhronizacija zapisa uređaja sa Apple-om na MDM server
* sinhronizacija "DEP profila" sa Apple-om sa MDM servera (koji se kasnije dostavlja uređaju od strane Apple-a)
* DEP "profil" sadrži:
* URL MDM vendor servera
* Dodatni pouzdani sertifikati za URL servera (opciono pinovanje)
* Dodatne postavke (npr. koje ekrane preskočiti u Setup Assistant-u)

## Serijski broj

Apple uređaji proizvedeni posle 2010. godine obično imaju **12-znakovne alfanumeričke** serijske brojeve, pri čemu **prva tri znaka predstavljaju lokaciju proizvodnje**, sledeća **dva** označavaju **godinu** i **nedelju** proizvodnje, sledeća **tri** znaka pružaju **jedinstveni identifikator**, a **poslednja** **četiri** znaka predstavljaju **broj modela**.

{% content-ref url="macos-serial-number.md" %}
[macos-serial-number.md](macos-serial-number.md)
{% endcontent-ref %}

[macos-serial-number.md](macos-serial-number.md) \{% endcontent-ref

### Korak 4: Provera DEP registracije - Dobijanje aktivacionog zapisa

Ovaj deo procesa se odvija kada **korisnik prvi put pokrene Mac** (ili nakon potpunog brisanja)

![](<../../../.gitbook/assets/image (568).png>)

ili kada se izvrši `sudo profiles show -type enrollment`

* Utvrditi da li je uređaj omogućen za DEP
* Aktivacioni zapis je interni naziv za DEP "profil"
* Počinje čim se uređaj poveže na internet
* Pokreće ga **`CPFetchActivationRecord`**
* Implementira ga **`cloudconfigurationd`** putem XPC-a. **"Setup Assistant"** (kada se uređaj prvi put pokrene) ili **`profiles`** komanda će kontaktirati ovaj daemon da bi preuzeli aktivacioni zapis.
* LaunchDaemon (uvek se pokreće kao root)

Sledi nekoliko koraka za dobijanje aktivacionog zapisa koji se izvodi pomoću **`MCTeslaConfigurationFetcher`**. Ovaj proces koristi enkripciju nazvanu **Absinthe**

1. Preuzimanje **sertifikata**
2. GET [https://iprofiles.apple.com/resource/certificate.cer](https://iprofiles.apple.com/resource/certificate.cer)
3. **Inicijalizacija** stanja iz sertifikata (**`NACInit`**)
4. Koristi različite podatke specifične za uređaj (npr. **serijski broj putem `IOKit`**)
5. Preuzimanje **sesijskog ključa**
6. POST [https://iprofiles.apple.com/session](https://iprofiles.apple.com/session)
7. Uspostavljanje sesije (**`NACKeyEstablishment`**)
8. Slanje zahteva
9. POST na [https://iprofiles.apple.com/macProfile](https://iprofiles.apple.com/macProfile) šaljući podatke `{ "action": "RequestProfileConfiguration", "sn": "" }`
10. JSON payload je enkriptovan pomoću Absinthe-a (**`NACSign`**)
11. Svi zahtevi se šalju preko HTTPs, koriste se ugrađeni korenski sertifikati

![](<../../../.gitbook/assets/image (566).png>)

Odgovor je JSON rečnik sa nekim važnim podacima kao što su:

* **url**: URL MDM dobavljačkog hosta za aktivacioni profil
* **anchor-certs**: Niz DER sertifikata koji se koriste kao pouzdani koreni

### **Korak 5: Preuzimanje profila**

![](<../../../.gitbook/assets/image (567).png>)

* Zahtev se šalje na **url koji je naveden u DEP profilu**.
* Ako su dostupni, koriste se **koreni sertifikati** za **proveru poverenja**.
* Podsetnik: svojstvo **anchor\_certs** DEP profila
* Zahtev je jednostavan .plist sa identifikacijom uređaja
* Primeri: **UDID, verzija OS-a**.
* Potpisano CMS-om, DER-kodirano
* Potpisano pomoću **sertifikata identiteta uređaja (iz APNS-a)**
* **Lanac sertifikata** uključuje istekli **Apple iPhone Device CA**

![](https://github.com/carlospolop/hacktricks/blob/rs/.gitbook/assets/image%20\(567\)%20\(1\)%20\(2\)%20\(2\)%20\(2\)%20\(2\)%20\(2\)%20\(2\)%20\(2\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(7\).png)

### Korak 6: Instalacija profila

* Nakon preuzimanja, **profil se čuva na sistemu**
* Ovaj korak se automatski pokreće (ako je u **pomoćniku za podešavanje**)
* Pokreće ga **`CPInstallActivationProfile`**
* Implementira ga mdmclient preko XPC-a
* LaunchDaemon (kao root) ili LaunchAgent (kao korisnik), zavisno od konteksta
* Konfiguracioni profili imaju više payloada za instalaciju
* Okvir ima arhitekturu zasnovanu na pluginima za instalaciju profila
* Svaki tip payloada je povezan sa pluginom
* Može biti XPC (u okviru) ili klasični Cocoa (u ManagedClient.app)
* Primer:
* Payloadi sertifikata koriste CertificateService.xpc

Tipično, **aktivacioni profil** koji pruža MDM dobavljač će **uključivati sledeće payloade**:

* `com.apple.mdm`: za **upisivanje** uređaja u MDM
* `com.apple.security.scep`: za bezbedno obezbeđivanje **klijentskog sertifikata** uređaju.
* `com.apple.security.pem`: za **instaliranje pouzdanih CA sertifikata** u sistemski ključni lanac uređaja.
* Instaliranje MDM payloada ekvivalentno **MDM proveri u dokumentaciji**
* Payload **sadrži ključna svojstva**:
*
* MDM Check-In URL (**`CheckInURL`**)
* MDM Command Polling URL (**`ServerURL`**) + APNs tema za pokretanje
* Za instaliranje MDM payloada, zahtev se šalje na **`CheckInURL`**
* Implementirano u **`mdmclient`**
* MDM payload može zavisiti od drugih payloada
* Omogućava **zahteve da budu vezani za određene sertifikate**:
* Svojstvo: **`CheckInURLPinningCertificateUUIDs`**
* Svojstvo: **`ServerURLPinningCertificateUUIDs`**
* Isporučeno putem PEM payloada
* Omogućava uređaju da bude povezan sa sertifikatom identiteta:
* Svojstvo: IdentityCertificateUUID
* Isporučeno putem SCEP payloada

### **Korak 7: Slušanje MDM komandi**

* Nakon što je MDM provera završena, dobavljač može **izdati push obaveštenja putem APNs-a**
* Po prijemu, obrađuje se pomoću **`mdmclient`**
* Za preuzimanje MDM komandi, zahtev se šalje na ServerURL
* Koristi se prethodno instalirani MDM payload:
* **`ServerURLPinningCertificateUUIDs`** za vezivanje zahteva
* **`IdentityCertificateUUID`** za TLS klijentski sertifikat

## Napadi

### Upisivanje uređaja u druge organizacije

Kao što je ranije navedeno, da bi se pokušalo upisivanje uređaja u organizaciju, potreban je **samo serijski broj koji pripada toj organizaciji**. Nakon što je uređaj upisan, nekoliko organizacija će instalirati osetljive podatke na novi uređaj: sertifikate, aplikacije, lozinke za WiFi, VPN konfiguracije [i tako dalje](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
Stoga, ovo može biti opasan ulaz za napadače ako proces upisa nije pravilno zaštićen:

{% content-ref url="enrolling-devices-in-other-organisations.md" %}
[enrolling-devices-in-other-organisations.md](enrolling-devices-in-other-organisations.md)
{% endcontent-ref %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju eks

</details>
