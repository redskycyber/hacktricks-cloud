# macOS Launch/Environment Constraints & Trust Cache

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>

## Osnovne informacije

Ograničenja pokretanja u macOS-u su uvedena radi poboljšanja bezbednosti tako što **regulišu kako, ko i odakle se može pokrenuti proces**. Uvedena u macOS Ventura, pružaju okvir koji kategorizuje **svaki sistemski binarni fajl u odvojene kategorije ograničenja**, koje su definisane unutar **keša poverenja**, liste koja sadrži sistemski binarni fajl i njegov odgovarajući heš. Ova ograničenja se odnose na svaki izvršni binarni fajl u sistemu i obuhvataju skup **pravila** koja definišu zahteve za **pokretanje određenog binarnog fajla**. Pravila obuhvataju ograničenja koja binarni fajl mora zadovoljiti, ograničenja roditeljskog procesa koja moraju biti ispunjena od strane roditeljskog procesa, kao i ograničenja odgovornosti koja moraju biti poštovana od strane drugih relevantnih entiteta.

Mehanizam se proširuje na aplikacije trećih strana putem **Ograničenja okruženja**, počevši od macOS Sonoma, omogućavajući programerima da zaštite svoje aplikacije specificiranjem **skupa ključeva i vrednosti za ograničenja okruženja**.

Definišete **ograničenja pokretanja okruženja i biblioteke** u rečnicima ograničenja koje čuvate u **`launchd` property list fajlovima**, ili u **posebnim property list fajlovima** koje koristite pri potpisivanju koda.

Postoje 4 vrste ograničenja:

* **Ograničenja samog procesa**: Ograničenja primenjena na **pokrenuti** binarni fajl.
* **Ograničenja roditeljskog procesa**: Ograničenja primenjena na **roditeljski proces** (na primer **`launchd`** koji pokreće XP servis).
* **Ograničenja odgovornosti**: Ograničenja primenjena na **proces koji poziva servis** u XPC komunikaciji.
* **Ograničenja učitavanja biblioteke**: Koristite ograničenja učitavanja biblioteke da biste selektivno opisali kod koji može biti učitan.

Dakle, kada proces pokuša da pokrene drugi proces - pozivajući `execve(_:_:_:)` ili `posix_spawn(_:_:_:_:_:_:)` - operativni sistem proverava da li **izvršni** fajl **zadovoljava** svoje **sopstveno ograničenje**. Takođe proverava da li **izvršni fajl roditeljskog procesa** zadovoljava ograničenje roditeljskog procesa izvršnog fajla, i da li **izvršni fajl odgovornog procesa** zadovoljava ograničenje odgovornog procesa izvršnog fajla. Ako neko od ovih ograničenja pokretanja nije ispunjeno, operativni sistem ne pokreće program.

Ako prilikom učitavanja biblioteke bilo koji deo **ograničenja biblioteke nije tačan**, vaš proces **neće učitati** biblioteku.

## LC Kategorije

LC se sastoji od **činjenica** i **logičkih operacija** (i, ili...) koje kombinuju činjenice.

[**Činjenice koje LC može koristiti su dokumentovane**](https://developer.apple.com/documentation/security/defining\_launch\_environment\_and\_library\_constraints). Na primer:

* is-init-proc: Boolean vrednost koja označava da li izvršni fajl mora biti inicijalni proces operativnog sistema (`launchd`).
* is-sip-protected: Boolean vrednost koja označava da li izvršni fajl mora biti fajl zaštićen od strane System Integrity Protection (SIP).
* `on-authorized-authapfs-volume:` Boolean vrednost koja označava da li je operativni sistem učitao izvršni fajl sa autorizovanog, autentifikovanog APFS volumena.
* `on-authorized-authapfs-volume`: Boolean vrednost koja označava da li je operativni sistem učitao izvršni fajl sa autorizovanog, autentifikovanog APFS volumena.
* Cryptexes volumen
* `on-system-volume:` Boolean vrednost koja označava da li je operativni sistem učitao izvršni fajl sa trenutno podignutog sistemskog volumena.
* Unutar /System...
* ...

Kada se Apple binarni fajl potpiše, **dodeljuje mu se LC kategorija** unutar **keša poverenja**.

* **iOS 16 LC kategorija** su [**reverzirane i dokumentovane ovde**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056).
* Trenutne **LC kategorije (macOS 14** - Somona) su reverzirane i njihovi [**opisi se mogu pronaći ovde**](https://gist.github.com/theevilbit/a6fef1e0397425a334d064f7b6e1be53).

Na primer, Kategorija 1 je:

```
Category 1:
Self Constraint: (on-authorized-authapfs-volume || on-system-volume) && launch-type == 1 && validation-category == 1
Parent Constraint: is-init-proc
```

* `(on-authorized-authapfs-volume || on-system-volume)`: Morate biti na System ili Cryptexes volumenu.
* `launch-type == 1`: Morate biti sistemski servis (plist u LaunchDaemons).
* `validation-category == 1`: Izvršna datoteka operativnog sistema.
* `is-init-proc`: Launchd

### Reversiranje LC kategorija

Imate više informacija [**ovde**](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/#reversing-constraints), ali u osnovi, one su definisane u **AMFI (AppleMobileFileIntegrity)**, tako da morate preuzeti Kernel Development Kit da biste dobili **KEXT**. Simboli koji počinju sa **`kConstraintCategory`** su **interesantni**. Izdvajanjem njih dobijate DER (ASN.1) kodiran tok koji ćete morati dekodirati pomoću [ASN.1 Decoder](https://holtstrom.com/michael/tools/asn1decoder.php) ili python-asn1 biblioteke i njenog `dump.py` skripta, [andrivet/python-asn1](https://github.com/andrivet/python-asn1/tree/master) koji će vam dati razumljiviji string.

## Ograničenja okruženja

Ovo su postavljena ograničenja okruženja konfigurisana u **aplikacijama trećih strana**. Razvijač može odabrati **činjenice** i **logičke operatore** koje će koristiti u svojoj aplikaciji kako bi ograničio pristup sebi.

Moguće je nabrojati ograničenja okruženja aplikacije pomoću:

```bash
codesign -d -vvvv app.app
```

## Trust Caches

U **macOS**-u postoje nekoliko kešova poverenja:

* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4`**
* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`**
* **`/System/Library/Security/OSLaunchPolicyData`**

A na iOS-u izgleda da se nalazi u **`/usr/standalone/firmware/FUD/StaticTrustCache.img4`**.

{% hint style="warning" %}
Na macOS-u koji se izvršava na uređajima Apple Silicon, ako Apple potpisani binarni fajl nije u kešu poverenja, AMFI će odbiti da ga učita.
{% endhint %}

### Enumeracija kešova poverenja

Prethodni fajlovi keša poverenja su u formatu **IMG4** i **IM4P**, pri čemu je IM4P sekcija za prenos podataka u formatu IMG4.

Možete koristiti [**pyimg4**](https://github.com/m1stadev/PyIMG4) da izvučete prenos podataka iz baza:

{% code overflow="wrap" %}
```bash
# Installation
python3 -m pip install pyimg4

# Extract payloads data
cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/BaseSystemTrustCache.img4 -p /tmp/BaseSystemTrustCache.im4p
pyimg4 im4p extract -i /tmp/BaseSystemTrustCache.im4p -o /tmp/BaseSystemTrustCache.data

cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/StaticTrustCache.img4 -p /tmp/StaticTrustCache.im4p
pyimg4 im4p extract -i /tmp/StaticTrustCache.im4p -o /tmp/StaticTrustCache.data

pyimg4 im4p extract -i /System/Library/Security/OSLaunchPolicyData -o /tmp/OSLaunchPolicyData.data
```
{% endcode %}

(Druga opcija može biti korišćenje alata [**img4tool**](https://github.com/tihmstar/img4tool), koji će raditi čak i na M1 čipu, čak i ako je verzija starija i za x86\_64 ako ga instalirate na odgovarajućim lokacijama).

Sada možete koristiti alat [**trustcache**](https://github.com/CRKatri/trustcache) da biste dobili informacije u čitljivom formatu:

```bash
# Install
wget https://github.com/CRKatri/trustcache/releases/download/v2.0/trustcache_macos_arm64
sudo mv ./trustcache_macos_arm64 /usr/local/bin/trustcache
xattr -rc /usr/local/bin/trustcache
chmod +x /usr/local/bin/trustcache

# Run
trustcache info /tmp/OSLaunchPolicyData.data | head
trustcache info /tmp/StaticTrustCache.data | head
trustcache info /tmp/BaseSystemTrustCache.data | head

version = 2
uuid = 35EB5284-FD1E-4A5A-9EFB-4F79402BA6C0
entry count = 969
0065fc3204c9f0765049b82022e4aa5b44f3a9c8 [none] [2] [1]
00aab02b28f99a5da9b267910177c09a9bf488a2 [none] [2] [1]
0186a480beeee93050c6c4699520706729b63eff [none] [2] [2]
0191be4c08426793ff3658ee59138e70441fc98a [none] [2] [3]
01b57a71112235fc6241194058cea5c2c7be3eb1 [none] [2] [2]
01e6934cb8833314ea29640c3f633d740fc187f2 [none] [2] [2]
020bf8c388deaef2740d98223f3d2238b08bab56 [none] [2] [3]
```

Keš poverenja prati sledeću strukturu, tako da je **LC kategorija četvrta kolona**.

```c
struct trust_cache_entry2 {
uint8_t cdhash[CS_CDHASH_LEN];
uint8_t hash_type;
uint8_t flags;
uint8_t constraintCategory;
uint8_t reserved0;
} __attribute__((__packed__));
```

Zatim, možete koristiti skriptu kao što je [**ova**](https://gist.github.com/xpn/66dc3597acd48a4c31f5f77c3cc62f30) da izvučete podatke.

Iz tih podataka možete proveriti aplikacije sa **vrednošću ograničenja pokretanja `0`**, koje nisu ograničene ([**proverite ovde**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056) za svaku vrednost).

## Mitigacije napada

Ograničenja pokretanja bi sprečila nekoliko starih napada tako što bi **osigurala da se proces ne izvršava u neočekivanim uslovima**: na primer, iz neočekivanih lokacija ili pozivanjem od strane neočekivanog roditeljskog procesa (ako samo launchd treba da ga pokrene).

Osim toga, ograničenja pokretanja takođe **smanjuju rizik od napada degradacijom**.

Međutim, ona ne sprečavaju uobičajene zloupotrebe XPC-a, ubacivanje koda u Electron ili ubacivanje dylib biblioteka bez provere (osim ako su poznati ID-ovi timova koji mogu učitavati biblioteke).

### Zaštita XPC demona

U Sonoma izdanju, značajan detalj je **konfiguracija odgovornosti** XPC servisa demona. XPC servis je odgovoran za sebe, za razliku od povezanog klijenta koji je odgovoran. Ovo je dokumentovano u izveštaju o povratnoj informaciji FB13206884. Ova postavka može delovati nedostatno, jer omogućava određene interakcije sa XPC servisom:

* **Pokretanje XPC servisa**: Ako se pretpostavi da je ovo greška, ova postavka ne dozvoljava pokretanje XPC servisa putem napadačkog koda.
* **Povezivanje sa aktivnim servisom**: Ako je XPC servis već pokrenut (možda aktiviran od strane originalne aplikacije), nema prepreka za povezivanje sa njim.

Iako bi implementacija ograničenja na XPC servis mogla biti korisna tako što bi **smanjila mogućnost napada**, to ne rešava osnovnu brigu. Osiguravanje sigurnosti XPC servisa suštinski zahteva **efikasnu validaciju povezanog klijenta**. To ostaje jedini način za ojačavanje sigurnosti servisa. Takođe, treba napomenuti da pomenuta konfiguracija odgovornosti trenutno funkcioniše, što možda nije u skladu sa namerenim dizajnom.

### Zaštita Electrona

Čak i ako je potrebno da se aplikacija **otvori pomoću LaunchService-a** (u ograničenjima roditelja), to se može postići korišćenjem **`open`** (koji može postaviti okružne promenljive) ili korišćenjem **Launch Services API-ja** (gde se mogu naznačiti okružne promenljive).

## Reference

* [https://youtu.be/f1HA5QhLQ7Y?t=24146](https://youtu.be/f1HA5QhLQ7Y?t=24146)
* [https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/)
* [https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/](https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/)
* [https://developer.apple.com/videos/play/wwdc2023/10266/](https://developer.apple.com/videos/play/wwdc2023/10266/)

<details>

<summary><strong>Naučite hakovanje AWS-a od početka do naprednog nivoa sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite da vidite **vašu kompaniju reklamiranu na HackTricks**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova** [**hacktricks repo-u**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo-u**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>
