# macOS Keychain

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Glavni Keychain-ovi

* **User Keychain** (`~/Library/Keychains/login.keycahin-db`), koji se koristi za čuvanje **korisničkih kredencijala** kao što su lozinke aplikacija, internet lozinke, korisnički generisani sertifikati, mrežne lozinke i korisnički generisani javni/privatni ključevi.
* **System Keychain** (`/Library/Keychains/System.keychain`), koji čuva **sistemski kredencijali** kao što su WiFi lozinke, sistemski root sertifikati, sistemski privatni ključevi i lozinke aplikacija sistema.

### Pristup lozinki Keychain-a

Ove datoteke, iako nemaju inherentnu zaštitu i mogu biti **preuzete**, su šifrovane i zahtevaju **korisničku lozinku u plaintext-u za dešifrovanje**. Alat kao što je [**Chainbreaker**](https://github.com/n0fate/chainbreaker) može se koristiti za dešifrovanje.

## Zaštita unosa u Keychain-u

### ACLs

Svaki unos u Keychain-u je regulisan **Access Control Listama (ACLs)** koje određuju ko može izvršiti različite akcije na unosu Keychain-a, uključujući:

* **ACLAuhtorizationExportClear**: Dozvoljava nosiocu da dobije čisti tekst tajne.
* **ACLAuhtorizationExportWrapped**: Dozvoljava nosiocu da dobije šifrovan čisti tekst sa drugom pruženom lozinkom.
* **ACLAuhtorizationAny**: Dozvoljava nosiocu da izvrši bilo koju akciju.

ACLs su dalje praćene **listom pouzdanih aplikacija** koje mogu izvršiti ove akcije bez upita. To može biti:

* **N`il`** (nije potrebna autorizacija, **svi su pouzdani**)
* Prazna **lista** (nije pouzdan niko)
* **Lista** specifičnih **aplikacija**.

Takođe, unos može sadržati ključ **`ACLAuthorizationPartitionID`**, koji se koristi za identifikaciju **teamid, apple** i **cdhash**.

* Ako je naveden **teamid**, tada da bi se **pristupio vrednosti unosa** bez **upita**, korišćena aplikacija mora imati **isti teamid**.
* Ako je naveden **apple**, aplikacija mora biti **potpisana** od strane **Apple-a**.
* Ako je naznačen **cdhash**, onda aplikacija mora imati određeni **cdhash**.

### Kreiranje unosa u Keychain-u

Kada se **novi unos** kreira koristeći **`Keychain Access.app`**, primenjuju se sledeća pravila:

* Sve aplikacije mogu šifrovati.
* **Nijedna aplikacija** ne može izvoziti/dešifrovati (bez upita korisnika).
* Sve aplikacije mogu videti proveru integriteta.
* Nijedna aplikacija ne može menjati ACLs.
* **PartitionID** je postavljen na **`apple`**.

Kada **aplikacija kreira unos u Keychain-u**, pravila su malo drugačija:

* Sve aplikacije mogu šifrovati.
* Samo **kreirajuća aplikacija** (ili bilo koja druga eksplicitno dodata aplikacija) može izvoziti/dešifrovati (bez upita korisnika).
* Sve aplikacije mogu videti proveru integriteta.
* Nijedna aplikacija ne može menjati ACLs.
* **PartitionID** je postavljen na **`teamid:[ovde_teamID]`**.

## Pristup Keychain-u

### `security`

```bash
# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S
```

### API-ji

{% hint style="success" %}
**Enumeracija i ispisivanje** tajni u **ključnom lancu** koje **neće generisati upit** mogu se obaviti pomoću alata [**LockSmith**](https://github.com/its-a-feature/LockSmith)
{% endhint %}

Izlistajte i dobijte **informacije** o svakom unosu u ključni lanac:

* API **`SecItemCopyMatching`** daje informacije o svakom unosu i postoje neki atributi koje možete postaviti prilikom korišćenja:
* **`kSecReturnData`**: Ako je tačno, pokušaće da dešifruje podatke (postavite na netačno da biste izbegli potencijalne upite)
* **`kSecReturnRef`**: Dobijte i referencu na stavku u ključnom lancu (postavite na tačno ako kasnije vidite da možete dešifrovati bez upita)
* **`kSecReturnAttributes`**: Dobijte metapodatke o unosima
* **`kSecMatchLimit`**: Koliko rezultata vratiti
* **`kSecClass`**: Kakav unos u ključnom lancu

Dobijte **ACL-ove** svakog unosa:

* Pomoću API-ja **`SecAccessCopyACLList`** možete dobiti **ACL za stavku u ključnom lancu**, i vratiće listu ACL-ova (poput `ACLAuhtorizationExportClear` i drugih prethodno pomenutih) gde svaka lista ima:
* Opis
* **Lista pouzdanih aplikacija**. To može biti:
* Aplikacija: /Applications/Slack.app
* Binarna datoteka: /usr/libexec/airportd
* Grupa: group://AirPort

Izvezi podatke:

* API **`SecKeychainItemCopyContent`** dobija tekstualni oblik
* API **`SecItemExport`** izvozi ključeve i sertifikate, ali može biti potrebno postaviti lozinke da biste izvezli sadržaj šifrovan

I ovo su **zahtevi** da biste mogli **izvoziti tajnu bez upita**:

* Ako je navedeno **1+ pouzdanih** aplikacija:
* Potrebne su odgovarajuće **autorizacije** (**`Nil`**, ili biti **deo** dozvoljene liste aplikacija u autorizaciji za pristup tajnim informacijama)
* Potrebno je da potpis koda odgovara **PartitionID**
* Potrebno je da potpis koda odgovara potpisu jedne **pouzdane aplikacije** (ili biti član odgovarajuće KeychainAccessGroup)
* Ako su **sve aplikacije pouzdane**:
* Potrebne su odgovarajuće **autorizacije**
* Potrebno je da potpis koda odgovara **PartitionID**
* Ako nema **PartitionID**, onda ovo nije potrebno

{% hint style="danger" %}
Dakle, ako je navedena **1 aplikacija**, morate **ubaciti kod u tu aplikaciju**.

Ako je navedeno **apple** u **partitionID**, možete pristupiti tome pomoću **`osascript`**, tako da sve što veruje svim aplikacijama sa apple u partitionID. **`Python`** takođe može biti korišćen za ovo.
{% endhint %}

### Dva dodatna atributa

* **Nevidljivo**: To je boolean oznaka za **sakrivanje** unosa iz aplikacije **UI** Keychain
* **Opšte**: Služi za čuvanje **metapodataka** (tako da NIJE ŠIFROVANO)
* Microsoft je čuvao sve osvežavajuće tokene za pristup osetljivim krajnjim tačkama u obliku običnog teksta.

## Reference

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
