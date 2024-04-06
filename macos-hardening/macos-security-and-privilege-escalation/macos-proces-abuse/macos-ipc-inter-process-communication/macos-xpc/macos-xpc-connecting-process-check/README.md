# macOS XPC Connecting Process Check

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Provera povezivanja procesa u XPC-u

Kada se uspostavi veza sa XPC servisom, server će proveriti da li je veza dozvoljena. Ovo su provere koje obično vrši:

1. Provera da li je povezani **proces potpisan Apple-ovim** sertifikatom (koji se dodeljuje samo od strane Apple-a).

* Ako ovo **nije verifikovano**, napadač može kreirati **lažni sertifikat** koji odgovara bilo kojoj drugoj proveri.

2. Provera da li je povezani proces potpisan **sertifikatom organizacije** (verifikacija ID-a tima).

* Ako ovo **nije verifikovano**, **bilo koji razvojni sertifikat** od Apple-a može se koristiti za potpisivanje i povezivanje sa servisom.

3. Provera da li povezani proces **sadrži odgovarajući bundle ID**.

* Ako ovo **nije verifikovano**, bilo koji alat **potpisan od iste organizacije** može se koristiti za interakciju sa XPC servisom.

4. (4 ili 5) Provera da li povezani proces ima **odgovarajući broj verzije softvera**.

* Ako ovo **nije verifikovano**, stari, nesigurni klijenti koji su podložni ubacivanju procesa mogu se koristiti za povezivanje sa XPC servisom čak i uz ostale provere.

5. (4 ili 5) Provera da li povezani proces ima ojačano izvršavanje sa opasnim privilegijama (poput onih koje omogućavaju učitavanje proizvoljnih biblioteka ili korišćenje DYLD env varijabli).
6. Ako ovo **nije verifikovano**, klijent može biti **podložan ubacivanju koda**.
7. Provera da li povezani proces ima **privilegiju** koja mu omogućava povezivanje sa servisom. Ovo se odnosi na Apple binarne fajlove.
8. **Verifikacija** se mora **bazirati** na **audit token-u klijenta** koji se povezuje, **umesto** na njegovom ID-u procesa (**PID**) jer prvo sprečava **napade ponovnom upotrebom PID-a**.

* Razvojni programeri **retko koriste** API poziv za audit token jer je **privatan**, pa Apple može **promeniti** to u bilo kom trenutku. Takođe, korišćenje privatnih API-ja nije dozvoljeno u aplikacijama Mac App Store-a.
* Ako se koristi metoda **`processIdentifier`**, može biti ranjiva
* Treba koristiti **`xpc_dictionary_get_audit_token`** umesto **`xpc_connection_get_audit_token`**, jer poslednja može biti [ranjiva u određenim situacijama](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/).

### Napadi na komunikaciju

Za više informacija o napadu ponovnom upotrebom PID-a pogledajte:

{% content-ref url="macos-pid-reuse.md" %}
[macos-pid-reuse.md](macos-pid-reuse.md)
{% endcontent-ref %}

Za više informacija o napadu **`xpc_connection_get_audit_token`** pogledajte:

{% content-ref url="macos-xpc_connection_get_audit_token-attack.md" %}
[macos-xpc\_connection\_get\_audit\_token-attack.md](macos-xpc\_connection\_get\_audit\_token-attack.md)
{% endcontent-ref %}

### Prevencija napada na Trustcache - Downgrade

Trustcache je defanzivna metoda koja je uvedena u Apple Silicon mašinama i čuva bazu podataka CDHSAH Apple binarnih fajlova, tako da se mogu izvršavati samo dozvoljeni, nepromenjeni binarni fajlovi. Ovo sprečava izvršavanje verzija sa nižim nivoom sigurnosti.

### Primeri koda

Server će implementirati ovu **verifikaciju** u funkciji nazvanoj **`shouldAcceptNewConnection`**.

{% code overflow="wrap" %}
```objectivec
- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
//Check connection
return YES;
}
```
{% endcode %}

Objekat NSXPCConnection ima **privatno** svojstvo **`auditToken`** (ono koje treba koristiti, ali se može promeniti) i **javno** svojstvo **`processIdentifier`** (ono koje ne treba koristiti).

Povezani proces može se proveriti na sledeći način:

{% code overflow="wrap" %}
```objectivec
[...]
SecRequirementRef requirementRef = NULL;
NSString requirementString = @"anchor apple generic and identifier \"xyz.hacktricks.service\" and certificate leaf [subject.CN] = \"TEAMID\" and info [CFBundleShortVersionString] >= \"1.0\"";
/* Check:
- Signed by a cert signed by Apple
- Check the bundle ID
- Check the TEAMID of the signing cert
- Check the version used
*/

// Check the requirements with the PID (vulnerable)
SecRequirementCreateWithString(requirementString, kSecCSDefaultFlags, &requirementRef);
SecCodeCheckValidity(code, kSecCSDefaultFlags, requirementRef);

// Check the requirements wuing the auditToken (secure)
SecTaskRef taskRef = SecTaskCreateWithAuditToken(NULL, ((ExtendedNSXPCConnection*)newConnection).auditToken);
SecTaskValidateForRequirement(taskRef, (__bridge CFStringRef)(requirementString))
```
{% endcode %}

Ako programer ne želi da proverava verziju klijenta, on može barem proveriti da klijent nije podložan ubacivanju procesa:

{% code overflow="wrap" %}
```objectivec
[...]
CFDictionaryRef csInfo = NULL;
SecCodeCopySigningInformation(code, kSecCSDynamicInformation, &csInfo);
uint32_t csFlags = [((__bridge NSDictionary *)csInfo)[(__bridge NSString *)kSecCodeInfoStatus] intValue];
const uint32_t cs_hard = 0x100;        // don't load invalid page.
const uint32_t cs_kill = 0x200;        // Kill process if page is invalid
const uint32_t cs_restrict = 0x800;    // Prevent debugging
const uint32_t cs_require_lv = 0x2000; // Library Validation
const uint32_t cs_runtime = 0x10000;   // hardened runtime
if ((csFlags & (cs_hard | cs_require_lv)) {
return Yes; // Accept connection
}
```
{% endcode %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
