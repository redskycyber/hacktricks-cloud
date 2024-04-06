# macOS IOKit

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite da vidite vašu **kompaniju reklamiranu na HackTricks-u**? Ili želite pristup **poslednjoj verziji PEASS-a ili preuzimanje HackTricks-a u PDF formatu**? Pogledajte [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu ekskluzivnu kolekciju [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS i HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) **Discord grupi** ili [**Telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Podelite svoje hakovanje trikova slanjem PR-a na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Osnovne informacije

IO Kit je open-source, objektno-orijentisani **framework za upravljače uređaja** u XNU kernelu, koji se bavi **dinamički učitanim upravljačima uređaja**. Omogućava dodavanje modularnog koda u kernel u hodu, podržavajući različiti hardver.

IOKit upravljači će u osnovi **izvoziti funkcije iz kernela**. Ovi tipovi parametara funkcija su **unapred definisani** i verifikovani. Osim toga, slično kao i XPC, IOKit je samo još jedan sloj **iznad Mach poruka**.

**IOKit XNU kernel kod** je otvoren od strane Apple-a na [https://github.com/apple-oss-distributions/xnu/tree/main/iokit](https://github.com/apple-oss-distributions/xnu/tree/main/iokit). Osim toga, IOKit komponente u korisničkom prostoru su takođe otvorenog koda [https://github.com/opensource-apple/IOKitUser](https://github.com/opensource-apple/IOKitUser).

Međutim, **nijedan IOKit upravljač** nije otvorenog koda. U svakom slučaju, povremeno se može pojaviti izdanje upravljača sa simbolima koji olakšavaju njegovo debagovanje. Pogledajte kako **dobiti ekstenzije upravljača iz firmware-a ovde**]\(./#ipsw)\*\*.

Napisan je u **C++**. Možete dobiti demangle C++ simbole sa:

```bash
# Get demangled symbols
nm -C com.apple.driver.AppleJPEGDriver

# Demangled symbols from stdin
c++filt
__ZN16IOUserClient202222dispatchExternalMethodEjP31IOExternalMethodArgumentsOpaquePK28IOExternalMethodDispatch2022mP8OSObjectPv
IOUserClient2022::dispatchExternalMethod(unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```

{% hint style="danger" %}
IOKit **izložene funkcije** mogu izvršiti **dodatne sigurnosne provjere** kada klijent pokuša pozvati funkciju, ali napomena da aplikacije obično su **ograničene** sandboxom s kojim IOKit funkcijama mogu komunicirati.
{% endhint %}

## Drajveri

U macOS-u se nalaze u:

* **`/System/Library/Extensions`**
* KEXT datoteke ugrađene u operativni sistem OS X.
* **`/Library/Extensions`**
* KEXT datoteke instalirane od strane softvera trećih strana.

U iOS-u se nalaze u:

* **`/System/Library/Extensions`**

```bash
#Use kextstat to print the loaded drivers
kextstat
Executing: /usr/bin/kmutil showloaded
No variant specified, falling back to release
Index Refs Address            Size       Wired      Name (Version) UUID <Linked Against>
1  142 0                  0          0          com.apple.kpi.bsd (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
2   11 0                  0          0          com.apple.kpi.dsep (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
3  170 0                  0          0          com.apple.kpi.iokit (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
4    0 0                  0          0          com.apple.kpi.kasan (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
5  175 0                  0          0          com.apple.kpi.libkern (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
6  154 0                  0          0          com.apple.kpi.mach (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
7   88 0                  0          0          com.apple.kpi.private (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
8  106 0                  0          0          com.apple.kpi.unsupported (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
9    2 0xffffff8003317000 0xe000     0xe000     com.apple.kec.Libm (1) 6C1342CC-1D74-3D0F-BC43-97D5AD38200A <5>
10   12 0xffffff8003544000 0x92000    0x92000    com.apple.kec.corecrypto (11.1) F5F1255F-6552-3CF4-A9DB-D60EFDEB4A9A <8 7 6 5 3 1>
```

Do broja 9 navedeni drajveri se **učitavaju na adresi 0**. To znači da oni nisu pravi drajveri već **deo jezgra i ne mogu se isključiti**.

Da biste pronašli određene ekstenzije, možete koristiti:

```bash
kextfind -bundle-id com.apple.iokit.IOReportFamily #Search by full bundle-id
kextfind -bundle-id -substring IOR #Search by substring in bundle-id
```

Da biste učitali i isključili kernel ekstenzije, uradite sledeće:

```bash
kextload com.apple.iokit.IOReportFamily
kextunload com.apple.iokit.IOReportFamily
```

## IORegistry

**IORegistry** je ključni deo IOKit okvira u macOS i iOS koji služi kao baza podataka za prikazivanje konfiguracije i stanja hardvera sistema. To je **hijerarhijska kolekcija objekata koja predstavlja sav hardver i drajvere** učitane na sistemu, kao i njihove međusobne odnose.

IORegistry možete dobiti koristeći CLI **`ioreg`** da biste ga pregledali iz konzole (posebno korisno za iOS).

```bash
ioreg -l #List all
ioreg -w 0 #Not cut lines
ioreg -p <plane> #Check other plane
```

Možete preuzeti **`IORegistryExplorer`** sa **Xcode Additional Tools** sa [**https://developer.apple.com/download/all/**](https://developer.apple.com/download/all/) i pregledati **macOS IORegistry** kroz **grafički** interfejs.

<figure><img src="../../../.gitbook/assets/image (695).png" alt="" width="563"><figcaption></figcaption></figure>

U IORegistryExplorer-u, "planes" se koriste za organizovanje i prikaz odnosa između različitih objekata u IORegistry-ju. Svaki plane predstavlja određenu vrstu odnosa ili određeni prikaz hardvera i konfiguracije drajvera sistema. Evo nekih uobičajenih planes-a sa kojima možete da se susretnete u IORegistryExplorer-u:

1. **IOService Plane**: Ovo je najopštiji plane, prikazuje objekte servisa koji predstavljaju drajvere i nub-ove (komunikacione kanale između drajvera). Prikazuje odnos između provajdera i klijenata ovih objekata.
2. **IODeviceTree Plane**: Ovaj plane predstavlja fizičke veze između uređaja kako su povezani sa sistemom. Često se koristi za vizualizaciju hijerarhije uređaja povezanih preko magistrala poput USB-a ili PCI-a.
3. **IOPower Plane**: Prikazuje objekte i njihove odnose u smislu upravljanja napajanjem. Može prikazati koji objekti utiču na stanje napajanja drugih, što je korisno za otklanjanje problema vezanih za napajanje.
4. **IOUSB Plane**: Posebno fokusiran na USB uređaje i njihove odnose, prikazuje hijerarhiju USB hubova i povezanih uređaja.
5. **IOAudio Plane**: Ovaj plane služi za prikazivanje audio uređaja i njihovih odnosa unutar sistema.
6. ...

## Primer koda za komunikaciju sa drajverom

Sledeći kod se povezuje sa IOKit servisom `"YourServiceNameHere"` i poziva funkciju unutar selektora 0. Za to:

* prvo se poziva **`IOServiceMatching`** i **`IOServiceGetMatchingServices`** da bi se dobio servis.
* Zatim se uspostavlja veza pozivom **`IOServiceOpen`**.
* Na kraju se poziva funkcija sa **`IOConnectCallScalarMethod`** koja označava selektor 0 (selektor je broj koji je dodeljen funkciji koju želite da pozovete).

```objectivec
#import <Foundation/Foundation.h>
#import <IOKit/IOKitLib.h>

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Get a reference to the service using its name
CFMutableDictionaryRef matchingDict = IOServiceMatching("YourServiceNameHere");
if (matchingDict == NULL) {
NSLog(@"Failed to create matching dictionary");
return -1;
}

// Obtain an iterator over all matching services
io_iterator_t iter;
kern_return_t kr = IOServiceGetMatchingServices(kIOMasterPortDefault, matchingDict, &iter);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to get matching services");
return -1;
}

// Get a reference to the first service (assuming it exists)
io_service_t service = IOIteratorNext(iter);
if (!service) {
NSLog(@"No matching service found");
IOObjectRelease(iter);
return -1;
}

// Open a connection to the service
io_connect_t connect;
kr = IOServiceOpen(service, mach_task_self(), 0, &connect);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to open service");
IOObjectRelease(service);
IOObjectRelease(iter);
return -1;
}

// Call a method on the service
// Assume the method has a selector of 0, and takes no arguments
kr = IOConnectCallScalarMethod(connect, 0, NULL, 0, NULL, NULL);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to call method");
}

// Cleanup
IOServiceClose(connect);
IOObjectRelease(service);
IOObjectRelease(iter);
}
return 0;
}
```

Postoje **druge** funkcije koje se mogu koristiti za pozivanje IOKit funkcija osim **`IOConnectCallScalarMethod`** kao što su **`IOConnectCallMethod`**, **`IOConnectCallStructMethod`**...

## Reversing driver entrypoint

Možete ih dobiti, na primer, iz [**firmware slike (ipsw)**](./#ipsw). Zatim je učitajte u svoj omiljeni dekompajler.

Možete početi dekompajlirati funkciju **`externalMethod`** jer je to funkcija drajvera koja će primati poziv i pozivati odgovarajuću funkciju:

<figure><img src="../../../.gitbook/assets/image (696).png" alt="" width="315"><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (697).png" alt=""><figcaption></figcaption></figure>

Taj užasan poziv znači:

```cpp
IOUserClient2022::dispatchExternalMethod(unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```

Primetite kako je u prethodnoj definiciji propustio parametar **`self`**, ispravna definicija bi bila:

{% code overflow="wrap" %}
```cpp
IOUserClient2022::dispatchExternalMethod(self, unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% endcode %}

Zapravo, pravu definiciju možete pronaći na [https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/Kernel/IOUserClient.cpp#L6388](https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/Kernel/IOUserClient.cpp#L6388):

```cpp
IOUserClient2022::dispatchExternalMethod(uint32_t selector, IOExternalMethodArgumentsOpaque *arguments,
const IOExternalMethodDispatch2022 dispatchArray[], size_t dispatchArrayCount,
OSObject * target, void * reference)
```

Sa ovim informacijama možete prepraviti Ctrl+Desno -> `Uredi potpis funkcije` i postaviti poznate tipove:

<figure><img src="../../../.gitbook/assets/image (702).png" alt=""><figcaption></figcaption></figure>

Novi dekompilirani kod će izgledati ovako:

<figure><img src="../../../.gitbook/assets/image (703).png" alt=""><figcaption></figcaption></figure>

Za sledeći korak trebamo definisati strukturu **`IOExternalMethodDispatch2022`**. To je open source na [https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/IOKit/IOUserClient.h#L168-L176](https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/IOKit/IOUserClient.h#L168-L176), možete je definisati:

<figure><img src="../../../.gitbook/assets/image (698).png" alt=""><figcaption></figcaption></figure>

Sada, prateći `(IOExternalMethodDispatch2022 *)&sIOExternalMethodArray` možete videti puno podataka:

<figure><img src="../../../.gitbook/assets/image (704).png" alt="" width="563"><figcaption></figcaption></figure>

Promenite tip podataka u **`IOExternalMethodDispatch2022:`**

<figure><img src="../../../.gitbook/assets/image (705).png" alt="" width="375"><figcaption></figcaption></figure>

nakon promene:

<figure><img src="../../../.gitbook/assets/image (707).png" alt="" width="563"><figcaption></figcaption></figure>

I kako sada znamo, tu imamo **niz od 7 elemenata** (proverite konačni dekompilirani kod), kliknite da biste napravili niz od 7 elemenata:

<figure><img src="../../../.gitbook/assets/image (708).png" alt="" width="563"><figcaption></figcaption></figure>

Nakon što je niz kreiran, možete videti sve izvezene funkcije:

<figure><img src="../../../.gitbook/assets/image (709).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Ako se sećate, da biste **pozvali** **izvezenu** funkciju iz korisničkog prostora, ne morate pozivati ime funkcije, već **broj selektora**. Ovde možete videti da je selektor **0** funkcija **`initializeDecoder`**, selektor **1** je **`startDecoder`**, selektor **2** je **`initializeEncoder`**...
{% endhint %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite pristup **poslednjoj verziji PEASS-a ili preuzimanje HackTricks-a u PDF formatu**? Pogledajte [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu ekskluzivnu kolekciju [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS i HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) **Discord grupi** ili [**Telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Podelite svoje hakovanje trikova slanjem PR-a** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
