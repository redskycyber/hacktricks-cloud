# macOS Library Injection

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

{% hint style="danger" %}
Kod **dyld je otvorenog koda** i može se naći na [https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/) i može se preuzeti kao tar koristeći **URL kao što je** [https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)
{% endhint %}

## **DYLD\_INSERT\_LIBRARIES**

Ovo je slično kao [**LD\_PRELOAD na Linux-u**](../../../../linux-hardening/privilege-escalation/#ld\_preload). Omogućava da se naznači proces koji će se pokrenuti da učita određenu biblioteku sa putanje (ako je omogućena env var)

Ova tehnika se takođe može **koristiti kao ASEP tehnika** jer svaka instalirana aplikacija ima plist koji se zove "Info.plist" koji omogućava **dodeljivanje okruženjskih promenljivih** koristeći ključ `LSEnvironmental`.

{% hint style="info" %}
Od 2012. godine **Apple je drastično smanjio moć** **`DYLD_INSERT_LIBRARIES`**.

Idite na kod i **proverite `src/dyld.cpp`**. U funkciji **`pruneEnvironmentVariables`** možete videti da su **`DYLD_*`** promenljive uklonjene.

U funkciji **`processRestricted`** postavljen je razlog ograničenja. Proverom tog koda možete videti da su razlozi:

* Binarni fajl je `setuid/setgid`
* Postojanje `__RESTRICT/__restrict` sekcije u macho binarnom fajlu.
* Softver ima privilegije (hardened runtime) bez [`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables) privilegije
* Proverite **privilegije** binarnog fajla sa: `codesign -dv --entitlements :- </path/to/bin>`

U ažuriranim verzijama ovu logiku možete pronaći u drugom delu funkcije **`configureProcessRestrictions`.** Međutim, ono što se izvršava u novijim verzijama su **početne provere funkcije** (možete ukloniti if-ove koji se odnose na iOS ili simulaciju jer se neće koristiti u macOS-u).
{% endhint %}

### Validacija biblioteke

Čak i ako binarni fajl omogućava korišćenje **`DYLD_INSERT_LIBRARIES`** env promenljive, ako binarni fajl proverava potpis biblioteke da bi je učitao, neće učitati prilagođenu biblioteku.

Da biste učitali prilagođenu biblioteku, binarni fajl mora imati **jednu od sledećih privilegija**:

* [`com.apple.security.cs.disable-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.security.cs.disable-library-validation)
* [`com.apple.private.security.clear-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.private.security.clear-library-validation)

ili binarni fajl **ne sme** imati **privilegiju hardened runtime** ili **privilegiju validacije biblioteke**.

Možete proveriti da li binarni fajl ima **privilegiju hardened runtime** sa `codesign --display --verbose <bin>` proveravajući privilegiju runtime u **`CodeDirectory`** kao: **`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

Takođe možete učitati biblioteku ako je **potpisana istim sertifikatom kao binarni fajl**.

Pronađite primer kako (zlo)upotrebiti ovo i proverite ograničenja u:

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dylib Hijacking

{% hint style="danger" %}
Zapamtite da se **prethodna ograničenja validacije biblioteke takođe primenjuju** na izvođenje napada Dylib hijacking.
{% endhint %}

Kao i u Windows-u, i na MacOS-u možete **hijackovati dylib** da biste naterali **aplikacije** da **izvršavaju** **proizvoljni** **kod** (dobro, zapravo od strane običnog korisnika ovo možda neće biti moguće jer možda će vam biti potrebna TCC dozvola da biste pisali unutar `.app` paketa i hijackovali biblioteku).\
Međutim, način na koji **MacOS** aplikacije **učitavaju** biblioteke je **strože ograničen** nego u Windows-u. To znači da **razvijači malvera** i dalje mogu koristiti ovu tehniku za **skrivanje**, ali verovatnoća da će moći **zloupotrebiti ovo za eskalaciju privilegija je mnogo manja**.

Prvo, **češće je** pronaći da **MacOS binarni fajlovi pokazuju punu putanju** do biblioteka koje treba učitati. I drugo, **MacOS nikada ne pretražuje** foldere **$PATH** za biblioteke.

**Glavni** deo **koda** koji se odnosi na ovu funkcionalnost je u **`ImageLoader::recursiveLoadLibraries`** u `ImageLoader.cpp`.

Postoje **4 različite komande zaglavlja** koje macho binarni fajl može koristiti za učitavanje biblioteka:

* Kom
* Ako **`LC_LOAD_DYLIB`** sadrži `@rpath/library.dylib` i **`LC_RPATH`** sadrži `/application/app.app/Contents/Framework/v1/` i `/application/app.app/Contents/Framework/v2/`. Obe fascikle će se koristiti za učitavanje `library.dylib`**.** Ako biblioteka ne postoji u `[...]/v1/` i napadač može da je postavi tamo da bi preuzeo učitavanje biblioteke u `[...]/v2/` jer se prate redosled putanja u **`LC_LOAD_DYLIB`**.
* **Pronađite putanje i biblioteke rpath** u binarnim fajlovima sa: `otool -l </path/to/binary> | grep -E "LC_RPATH|LC_LOAD_DYLIB" -A 5`

{% hint style="info" %}
**`@executable_path`**: Je **putanja** do direktorijuma koji sadrži **glavni izvršni fajl**.

**`@loader_path`**: Je **putanja** do **direktorijuma** koji sadrži **Mach-O binarni fajl** koji sadrži komandu za učitavanje.

* Kada se koristi u izvršnom fajlu, **`@loader_path`** je efektivno **isto** kao i **`@executable_path`**.
* Kada se koristi u **dylib**-u, **`@loader_path`** daje **putanju** do **dylib**-a.
{% endhint %}

Način za **povećanje privilegija** zloupotrebom ove funkcionalnosti bi bio u retkom slučaju kada **aplikacija** koju izvršava **root** traži neku **biblioteku u nekoj fascikli gde napadač ima dozvole za pisanje**.

{% hint style="success" %}
Dobar **skener** za pronalaženje **nedostajućih biblioteka** u aplikacijama je [**Dylib Hijack Scanner**](https://objective-see.com/products/dhs.html) ili [**CLI verzija**](https://github.com/pandazheng/DylibHijack).\
Dobar **izveštaj sa tehničkim detaljima** o ovoj tehnici može se naći [**ovde**](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x).
{% endhint %}

**Primer**

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dlopen Hijacking

{% hint style="danger" %}
Zapamtite da se **prethodna ograničenja provere biblioteka** takođe primenjuju na izvođenje napada Dlopen hijacking.
{% endhint %}

Iz **`man dlopen`**:

* Kada putanja **ne sadrži znak kosog crte** (tj. samo je ime lista), **dlopen() će vršiti pretragu**. Ako je **`$DYLD_LIBRARY_PATH`** postavljen pri pokretanju, dyld će prvo **tražiti u tom direktorijumu**. Zatim, ako pozivajući mach-o fajl ili glavni izvršni fajl specificira **`LC_RPATH`**, dyld će **tražiti u tim** direktorijumima. Zatim, ako je proces **neograničen**, dyld će tražiti u **trenutnom radnom direktorijumu**. Na kraju, za stare binarne fajlove, dyld će pokušati neke rezervne opcije. Ako je **`$DYLD_FALLBACK_LIBRARY_PATH`** postavljen pri pokretanju, dyld će tražiti u **tim direktorijumima**, inače će dyld tražiti u **`/usr/local/lib/`** (ako je proces neograničen), a zatim u **`/usr/lib/`** (ove informacije su preuzete iz **`man dlopen`**).

1. `$DYLD_LIBRARY_PATH`
2. `LC_RPATH`
3. `CWD` (ako je neograničen)
4. `$DYLD_FALLBACK_LIBRARY_PATH`
5. `/usr/local/lib/` (ako je neograničen)
6. `/usr/lib/`

{% hint style="danger" %}
Ako nema kosih crta u imenu, postoji 2 načina za izvršenje hijackinga:

* Ako je bilo koji **`LC_RPATH`** **upisiv** (ali se proverava potpis, tako da za ovo takođe treba da binarni fajl bude neograničen)
* Ako je binarni fajl **neograničen** i onda je moguće učitati nešto iz CWD (ili zloupotrebiti jednu od pomenutih env promenljivih)
{% endhint %}

* Kada putanja **izgleda kao putanja do framework-a** (npr. `/stuff/foo.framework/foo`), ako je **`$DYLD_FRAMEWORK_PATH`** postavljen pri pokretanju, dyld će prvo tražiti u tom direktorijumu za **delimičnu putanju framework-a** (npr. `foo.framework/foo`). Zatim, dyld će pokušati **datu putanju onakvu kakva je** (koristeći trenutni radni direktorijum za relativne putanje). Na kraju, za stare binarne fajlove, dyld će pokušati neke rezervne opcije. Ako je **`$DYLD_FALLBACK_FRAMEWORK_PATH`** postavljen pri pokretanju, dyld će tražiti u tim direktorijumima. Inače, tražiće u **`/Library/Frameworks`** (na macOS-u ako je proces neograničen), a zatim u **`/System/Library/Frameworks`**.

1. `$DYLD_FRAMEWORK_PATH`
2. data putanja (koristeći trenutni radni direktorijum za relativne putanje ako je neograničen)
3. `$DYLD_FALLBACK_FRAMEWORK_PATH`
4. `/Library/Frameworks` (ako je neograničen)
5. `/System/Library/Frameworks`

{% hint style="danger" %}
Ako je putanja framework-a, način za izvršenje hijackinga bi bio:

* Ako je proces **neograničen**, zloupotrebom **relativne putanje iz CWD** ili pomenutih env promenljivih (čak i ako nije navedeno u dokumentaciji, ako je proces ograničen DYLD\_\* env promenljive se uklanjaju)
{% endhint %}

* Kada putanja **sadrži kosu crtu ali nije putanja do framework-a** (tj. potpuna putanja ili delimična putanja do dylib-a), dlopen() prvo traži (ako je postavljeno) u **`$DYLD_LIBRARY_PATH`** (sa delom lista iz putanje). Zatim, dyld **pokušava datu putanju** (koristeći trenutni radni direktorijum za relativne putanje (ali samo za neograničene procese)). Na kraju, za stare binarne fajlove, dyld će pokušati neke rezervne opcije. Ako je **`$DYLD_FALLBACK_LIBRARY_PATH`** postavljen pri pokretanju, dyld će tražiti u tim direktorijumima, inače će dyld tražiti u **`/usr/local/lib/`** (ako je proces neograničen), a zatim u **`/usr/lib/`**.

1. `$DYLD_LIBRARY_PATH`
2. data putanja (koristeći trenutni radni direktorijum za relativne putanje ako je neograničen)
3. `$DYLD_FALLBACK_LIBRARY_PATH`
4. `/usr/local/lib/` (ako je neograničen)
5. `/usr/lib/`

{% hint style="danger" %}
Ako ima kosih crta u imenu i nije putanja do framework-a, način za izvršenje hijackinga bi bio:

* Ako je binarni fajl **neograničen** i onda je moguće učitati nešto iz CWD ili `/usr/local/lib` (ili zloupotrebiti jednu od pomenutih env promenljivih)
{% endhint %}

Napomena: Ne postoje konfiguracioni fajlovi za kontrolu pretrage dlopen.

Napomena: Ako je glavni izvršni fajl **set\[ug]id binarni fajl ili potpisan sa privilegijama**, tada se **sve env promenljive ignorišu**, i može se koristiti samo potpuna putanja (\[proverite ograničenja DYLD\_INSERT\_

```c
// gcc dlopentest.c -o dlopentest -Wl,-rpath,/tmp/test
#include <dlfcn.h>
#include <stdio.h>

int main(void)
{
void* handle;

fprintf("--- No slash ---\n");
handle = dlopen("just_name_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative framework ---\n");
handle = dlopen("a/framework/rel_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs framework ---\n");
handle = dlopen("/a/abs/framework/abs_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative Path ---\n");
handle = dlopen("a/folder/rel_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs Path ---\n");
handle = dlopen("/a/abs/folder/abs_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

return 0;
}
```

Ako ga kompajlirate i izvršite, možete videti **gde je svaka biblioteka neuspešno pretražena**. Takođe, možete **filtrirati FS logove**:

```bash
sudo fs_usage | grep "dlopentest"
```

## Zloupotreba relativne putanje

Ako je **privilegovan binarni fajl/aplikacija** (poput SUID-a ili nekog binarnog fajla sa moćnim ovlašćenjima) koji **učitava relativnu putanju** biblioteke (na primer koristeći `@executable_path` ili `@loader_path`) i ima onemogućenu **proveru biblioteke**, moguće je premestiti binarni fajl na lokaciju gde napadač može **izmeniti relativnu putanju učitane biblioteke** i iskoristiti je za ubacivanje koda u proces.

## Uklanjanje `DYLD_*` i `LD_LIBRARY_PATH` env promenljivih

U fajlu `dyld-dyld-832.7.1/src/dyld2.cpp` moguće je pronaći funkciju **`pruneEnvironmentVariables`**, koja će ukloniti sve env promenljive koje **počinju sa `DYLD_`** i **`LD_LIBRARY_PATH=`**.

Takođe će postaviti na **null** specifične env promenljive **`DYLD_FALLBACK_FRAMEWORK_PATH`** i **`DYLD_FALLBACK_LIBRARY_PATH`** za **suid** i **sgid** binarne fajlove.

Ova funkcija se poziva iz **`_main`** funkcije istog fajla ako je ciljani operativni sistem OSX, na sledeći način:

```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```

i te boolean zastavice se postavljaju u istoj datoteci u kodu:

```cpp
#if TARGET_OS_OSX
// support chrooting from old kernel
bool isRestricted = false;
bool libraryValidation = false;
// any processes with setuid or setgid bit set or with __RESTRICT segment is restricted
if ( issetugid() || hasRestrictedSegment(mainExecutableMH) ) {
isRestricted = true;
}
bool usingSIP = (csr_check(CSR_ALLOW_TASK_FOR_PID) != 0);
uint32_t flags;
if ( csops(0, CS_OPS_STATUS, &flags, sizeof(flags)) != -1 ) {
// On OS X CS_RESTRICT means the program was signed with entitlements
if ( ((flags & CS_RESTRICT) == CS_RESTRICT) && usingSIP ) {
isRestricted = true;
}
// Library Validation loosens searching but requires everything to be code signed
if ( flags & CS_REQUIRE_LV ) {
isRestricted = false;
libraryValidation = true;
}
}
gLinkContext.allowAtPaths                = !isRestricted;
gLinkContext.allowEnvVarsPrint           = !isRestricted;
gLinkContext.allowEnvVarsPath            = !isRestricted;
gLinkContext.allowEnvVarsSharedCache     = !libraryValidation || !usingSIP;
gLinkContext.allowClassicFallbackPaths   = !isRestricted;
gLinkContext.allowInsertFailures         = false;
gLinkContext.allowInterposing         	 = true;
```

Što zapravo znači da ako je binarni fajl **suid** ili **sgid**, ili ima **RESTRICT** segment u zaglavljima ili je potpisan sa **CS\_RESTRICT** zastavicom, tada je **`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`** tačno i okružne promenljive su uklonjene.

Imajte na umu da ako je CS\_REQUIRE\_LV tačno, tada promenljive neće biti uklonjene, ali će se proveriti validacija biblioteke da koriste isti sertifikat kao i originalni binarni fajl.

## Provera ograničenja

### SUID & SGID

```bash
# Make it owned by root and suid
sudo chown root hello
sudo chmod +s hello
# Insert the library
DYLD_INSERT_LIBRARIES=inject.dylib ./hello

# Remove suid
sudo chmod -s hello
```

### Sekcija `__RESTRICT` sa segmentom `__restrict`

***

#### Description

The `__RESTRICT` section is a segment in macOS that is used to restrict the loading of dynamic libraries. This section is designed to prevent library injection attacks, where malicious code is injected into a legitimate process by loading a malicious library.

***

#### Opis

Sekcija `__RESTRICT` je segment u macOS-u koji se koristi za ograničavanje učitavanja dinamičkih biblioteka. Ova sekcija je dizajnirana da spreči napade ubacivanja biblioteka, gde se zlonamerni kod ubacuje u legitimni proces učitavanjem zlonamerne biblioteke.

```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```

### Ojačani runtime

Kreirajte novi sertifikat u Keychain-u i koristite ga za potpisivanje binarnog fajla:

{% code overflow="wrap" %}
```bash
# Apply runtime proetction
codesign -s <cert-name> --option=runtime ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello #Library won't be injected

# Apply library validation
codesign -f -s <cert-name> --option=library ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed #Will throw an error because signature of binary and library aren't signed by same cert (signs must be from a valid Apple-signed developer certificate)

# Sign it
## If the signature is from an unverified developer the injection will still work
## If it's from a verified developer, it won't
codesign -f -s <cert-name> inject.dylib
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed

# Apply CS_RESTRICT protection
codesign -f -s <cert-name> --option=restrict hello-signed
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed # Won't work
```
{% endcode %}

{% hint style="danger" %}
Imajte na umu da čak i ako postoje binarni fajlovi potpisani sa zastavicama **`0x0(none)`**, mogu dobiti dinamički zastavicu **`CS_RESTRICT`** prilikom izvršavanja i zbog toga ova tehnika neće raditi na njima.

Možete proveriti da li proces ima ovu zastavicu sa (preuzmite [**csops ovde**](https://github.com/axelexic/CSOps)):

```bash
csops -status <pid>
```

zatim proverite da li je zastavica 0x800 omogućena.
{% endhint %}

## Reference

* [https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/](https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
