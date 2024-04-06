# AD CS Certificate Theft

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Ovo je kratak rezime poglavlja o krađi sertifikata iz impresivnog istraživanja sa** [**https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf**](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)

## Šta mogu da uradim sa sertifikatom

Pre nego što proverimo kako ukrasti sertifikate, evo nekih informacija o tome za šta je sertifikat koristan:

```powershell
# Powershell
$CertPath = "C:\path\to\cert.pfx"
$CertPass = "P@ssw0rd"
$Cert = New-Object
System.Security.Cryptography.X509Certificates.X509Certificate2 @($CertPath, $CertPass)
$Cert.EnhancedKeyUsageList

# cmd
certutil.exe -dump -v cert.pfx
```

## Izvoz sertifikata korišćenjem Crypto API-ja - KRAĐA1

U **interaktivnoj sesiji radne površine**, izdvajanje korisničkog ili mašinskog sertifikata, zajedno sa privatnim ključem, može se lako izvršiti, posebno ako je **privatni ključ izvoziv**. To se može postići tako što se navigira do sertifikata u `certmgr.msc`, desnim klikom na njega i odabirom `All Tasks → Export` da bi se generisao zaštićeni lozinkom .pfx fajl.

Za **programski pristup**, dostupni su alati poput PowerShell `ExportPfxCertificate` cmdleta ili projekti poput [TheWover-ovog CertStealer C# projekta](https://github.com/TheWover/CertStealer). Oni koriste **Microsoft CryptoAPI** (CAPI) ili Cryptography API: Next Generation (CNG) za interakciju sa skladištem sertifikata. Ovi API-ji pružaju niz kriptografskih usluga, uključujući one neophodne za skladištenje i autentifikaciju sertifikata.

Međutim, ako je privatni ključ postavljen kao neizvoziv, kako CAPI tako i CNG će obično blokirati izdvajanje takvih sertifikata. Da bi se zaobišlo ovo ograničenje, mogu se koristiti alati poput **Mimikatz**-a. Mimikatz nudi komande `crypto::capi` i `crypto::cng` za zakrpu odgovarajućih API-ja, omogućavajući izvoz privatnih ključeva. Konkretno, `crypto::capi` zakrpljuje CAPI unutar trenutnog procesa, dok `crypto::cng` cilja memoriju **lsass.exe**-a za zakrpu.

## Krađa korisničkog sertifikata putem DPAPI-ja - KRAĐA2

Više informacija o DPAPI-ju možete pronaći u:

{% content-ref url="../../windows-local-privilege-escalation/dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](../../windows-local-privilege-escalation/dpapi-extracting-passwords.md)
{% endcontent-ref %}

U Windows-u, **privatni ključevi sertifikata su zaštićeni DPAPI-jem**. Važno je prepoznati da su **lokacije skladištenja korisničkih i mašinskih privatnih ključeva** različite, a strukture fajlova se razlikuju u zavisnosti od kriptografskog API-ja koji se koristi u operativnom sistemu. **SharpDPAPI** je alat koji može automatski navigirati kroz ove razlike prilikom dešifrovanja DPAPI blokova.

**Korisnički sertifikati** se uglavnom nalaze u registru pod `HKEY_CURRENT_USER\SOFTWARE\Microsoft\SystemCertificates`, ali neki se mogu naći i u direktorijumu `%APPDATA%\Microsoft\SystemCertificates\My\Certificates`. Odgovarajući **privatni ključevi** za ove sertifikate obično se skladište u `%APPDATA%\Microsoft\Crypto\RSA\User SID\` za **CAPI** ključeve i `%APPDATA%\Microsoft\Crypto\Keys\` za **CNG** ključeve.

Da biste **izdvojili sertifikat i njegov pripadajući privatni ključ**, proces uključuje:

1. **Odabir ciljnog sertifikata** iz korisnikovog skladišta i dobijanje imena njegovog skladišta ključeva.
2. **Lociranje potrebnog DPAPI master ključa** za dešifrovanje odgovarajućeg privatnog ključa.
3. **Dešifrovanje privatnog ključa** korišćenjem plaintext DPAPI master ključa.

Za **dobijanje plaintext DPAPI master ključa**, mogu se koristiti sledeći pristupi:

```bash
# With mimikatz, when running in the user's context
dpapi::masterkey /in:"C:\PATH\TO\KEY" /rpc

# With mimikatz, if the user's password is known
dpapi::masterkey /in:"C:\PATH\TO\KEY" /sid:accountSid /password:PASS
```

Da bi se olakšalo dešifrovanje fajlova master ključeva i fajlova privatnih ključeva, korisna je komanda `certificates` iz [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI). Prihvata `/pvk`, `/mkfile`, `/password` ili `{GUID}:KEY` kao argumente za dešifrovanje privatnih ključeva i povezanih sertifikata, čime se generiše `.pem` fajl.

```bash
# Decrypting using SharpDPAPI
SharpDPAPI.exe certificates /mkfile:C:\temp\mkeys.txt

# Converting .pem to .pfx
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

## Krađa mašinskog sertifikata putem DPAPI – THEFT3

Mašinski sertifikati koje Windows čuva u registru na putanji `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\SystemCertificates`, kao i pripadajući privatni ključevi smešteni na lokacijama `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\RSA\MachineKeys` (za CAPI) i `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\Keys` (za CNG), šifruju se pomoću DPAPI master ključeva mašine. Ovi ključevi se ne mogu dešifrovati pomoću rezervnog DPAPI ključa domena; umesto toga, potreban je **DPAPI\_SYSTEM LSA tajni ključ**, do kojeg samo korisnik SYSTEM može da pristupi.

Ručno dešifrovanje se može postići izvršavanjem komande `lsadump::secrets` u alatu **Mimikatz** kako bi se izvukao DPAPI\_SYSTEM LSA tajni ključ, a zatim se koristi ovaj ključ za dešifrovanje mašinskih master ključeva. Alternativno, komanda `crypto::certificates /export /systemstore:LOCAL_MACHINE` u alatu Mimikatz može se koristiti nakon zakrpe CAPI/CNG kao što je prethodno opisano.

**SharpDPAPI** nudi automatizovaniji pristup sa svojom komandom certificates. Kada se koristi zastavica `/machine` sa privilegijama podignutim na SYSTEM, on prelazi na SYSTEM, izbacuje DPAPI\_SYSTEM LSA tajni ključ, koristi ga za dešifrovanje mašinskih DPAPI master ključeva, a zatim koristi ove ključeve u obliku tekstualne tabele za dešifrovanje bilo kojih privatnih ključeva mašinskog sertifikata.

## Pronalaženje fajlova sertifikata – THEFT4

Sertifikati se ponekad nalaze direktno u fajl sistemu, kao što su deljeni fajlovi ili fascikla "Downloads". Najčešće korišćeni tipovi fajlova sertifikata u Windows okruženjima su `.pfx` i `.p12` fajlovi. Iako ređe, pojavljuju se i fajlovi sa ekstenzijama `.pkcs12` i `.pem`. Dodatne značajne ekstenzije fajlova povezanih sa sertifikatima uključuju:

* `.key` za privatne ključeve,
* `.crt`/`.cer` za samo sertifikate,
* `.csr` za zahteve za potpisivanje sertifikata koji ne sadrže sertifikate ili privatne ključeve,
* `.jks`/`.keystore`/`.keys` za Java keystore-ove, koji mogu sadržati sertifikate zajedno sa privatnim ključevima koji se koriste u Java aplikacijama.

Ove fajlove možete pretraživati pomoću PowerShell-a ili komandne linije tako što ćete tražiti pomenute ekstenzije.

U slučajevima kada se pronađe PKCS#12 fajl sertifikata koji je zaštićen lozinkom, moguće je izvući heš pomoću alata `pfx2john.py`, dostupnog na [fossies.org](https://fossies.org/dox/john-1.9.0-jumbo-1/pfx2john\_8py\_source.html). Nakon toga, može se koristiti JohnTheRipper za pokušaj pucanja lozinke.

```powershell
# Example command to search for certificate files in PowerShell
Get-ChildItem -Recurse -Path C:\Users\ -Include *.pfx, *.p12, *.pkcs12, *.pem, *.key, *.crt, *.cer, *.csr, *.jks, *.keystore, *.keys

# Example command to use pfx2john.py for extracting a hash from a PKCS#12 file
pfx2john.py certificate.pfx > hash.txt

# Command to crack the hash with JohnTheRipper
john --wordlist=passwords.txt hash.txt
```

## Krađa NTLM akreditacija putem PKINIT-a - THEFT5

Dati sadržaj objašnjava metod za krađu NTLM akreditacija putem PKINIT-a, posebno kroz metodu krađe označenu kao THEFT5. Evo ponovnog objašnjenja u pasivnom glasu, sa anonimizovanim sadržajem i sažetim gde je to moguće:

Da bi podržao NTLM autentifikaciju \[MS-NLMP] za aplikacije koje ne omogućavaju Kerberos autentifikaciju, KDC je dizajniran da vrati NTLM jednosmernu funkciju (OWF) korisnika unutar privilegovanog atributnog sertifikata (PAC), tačnije u baferu `PAC_CREDENTIAL_INFO`, kada se koristi PKCA. Kao rezultat toga, ukoliko se nalog autentifikuje i obezbedi Ticket-Granting Ticket (TGT) putem PKINIT-a, mehanizam je inherentno omogućen koji omogućava trenutnom hostu da izvuče NTLM heš iz TGT-a kako bi podržao zastarele autentifikacione protokole. Ovaj proces podrazumeva dešifrovanje strukture `PAC_CREDENTIAL_DATA`, koja je suštinski NDR serijalizovani prikaz NTLM plaintexta.

Alat **Kekeo**, dostupan na [https://github.com/gentilkiwi/kekeo](https://github.com/gentilkiwi/kekeo), se pominje kao sposoban da zahteva TGT koji sadrži ove specifične podatke, čime se omogućava dobijanje NTLM korisnika. Komanda koja se koristi u tu svrhu je sledeća:

```bash
tgt::pac /caname:generic-DC-CA /subject:genericUser /castore:current_user /domain:domain.local
```

Dodatno, napominje se da Kekeo može obraditi sertifikate zaštićene pametnim karticama, pod uslovom da se pin može dobiti, uz referencu na [https://github.com/CCob/PinSwipe](https://github.com/CCob/PinSwipe). Ista mogućnost je naznačena da je podržana i od strane **Rubeus**-a, dostupnog na [https://github.com/GhostPack/Rubeus](https://github.com/GhostPack/Rubeus).

Ovo objašnjenje obuhvata proces i alate koji su uključeni u krađu NTLM akreditiva putem PKINIT-a, fokusirajući se na dobijanje NTLM heševa putem TGT-a dobijenog korišćenjem PKINIT-a, kao i na alate koji olakšavaju ovaj proces.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
