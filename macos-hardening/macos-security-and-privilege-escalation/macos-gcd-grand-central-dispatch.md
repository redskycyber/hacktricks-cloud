# macOS GCD - Grand Central Dispatch

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovne informacije

**Grand Central Dispatch (GCD)**, takođe poznat kao **libdispatch**, dostupan je i na macOS-u i iOS-u. To je tehnologija koju je Apple razvio kako bi optimizovao podršku aplikacija za istovremeno (višenitno) izvršavanje na višejezgarnom hardveru.

**GCD** obezbeđuje i upravlja **FIFO redovima** na koje vaša aplikacija može **predati zadatke** u obliku **blok objekata**. Blokovi predati redovima za raspodelu se **izvršavaju na skupu niti** koji je u potpunosti upravljan od strane sistema. GCD automatski kreira niti za izvršavanje zadataka u redovima za raspodelu i raspoređuje te zadatke da se izvrše na dostupnim jezgrima.

{% hint style="success" %}
Ukratko, da bi izvršili kod **paralelno**, procesi mogu slati **blokove koda GCD-u**, koji će se pobrinuti za njihovo izvršavanje. Stoga, procesi ne stvaraju nove niti; **GCD izvršava dati kod sa svojim sopstvenim skupom niti**.
{% endhint %}

Ovo je veoma korisno za uspešno upravljanje paralelnim izvršavanjem, smanjujući značajno broj niti koje procesi stvaraju i optimizujući paralelno izvršavanje. Ovo je idealno za zadatke koji zahtevaju **veliku paralelnost** (brute-forcing?) ili za zadatke koji ne smeju blokirati glavnu nit: Na primer, glavna nit na iOS-u upravlja interakcijama sa korisničkim interfejsom, pa se na ovaj način upravlja svaka druga funkcionalnost koja bi mogla da uspori aplikaciju (pretraga, pristup vebu, čitanje fajla...).

## Objective-C

U Objective-C-u postoje različite funkcije za slanje bloka koji će se izvršiti paralelno:

* [**dispatch\_async**](https://developer.apple.com/documentation/dispatch/1453057-dispatch\_async): Predaje blok za asinhrono izvršavanje na red za raspodelu i odmah se vraća.
* [**dispatch\_sync**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync): Predaje blok objekat za izvršavanje i vraća se nakon što se taj blok završi sa izvršavanjem.
* [**dispatch\_once**](https://developer.apple.com/documentation/dispatch/1447169-dispatch\_once): Izvršava blok objekat samo jednom tokom trajanja aplikacije.
* [**dispatch\_async\_and\_wait**](https://developer.apple.com/documentation/dispatch/3191901-dispatch\_async\_and\_wait): Predaje radnu stavku za izvršavanje i vraća se tek nakon što se završi izvršavanje. Za razliku od [**`dispatch_sync`**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync), ova funkcija poštuje sve atribute reda kada izvršava blok.

Ove funkcije očekuju sledeće parametre: [**`dispatch_queue_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_queue\_t) **`queue,`** [**`dispatch_block_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_block\_t) **`block`**

Ovo je **struktura Bloka**:

```c
struct Block {
void *isa; // NSConcreteStackBlock,...
int flags;
int reserved;
void *invoke;
struct BlockDescriptor *descriptor;
// captured variables go here
};
```

I ovo je primer za korišćenje **paralelizma** sa **`dispatch_async`**:

```objective-c
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_async(queue, ^{
    // Code to be executed in parallel
});
```

```objectivec
#import <Foundation/Foundation.h>

// Define a block
void (^backgroundTask)(void) = ^{
// Code to be executed in the background
for (int i = 0; i < 10; i++) {
NSLog(@"Background task %d", i);
sleep(1);  // Simulate a long-running task
}
};

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Create a dispatch queue
dispatch_queue_t backgroundQueue = dispatch_queue_create("com.example.backgroundQueue", NULL);

// Submit the block to the queue for asynchronous execution
dispatch_async(backgroundQueue, backgroundTask);

// Continue with other work on the main queue or thread
for (int i = 0; i < 10; i++) {
NSLog(@"Main task %d", i);
sleep(1);  // Simulate a long-running task
}
}
return 0;
}
```

## Swift

**`libswiftDispatch`** je biblioteka koja pruža **Swift veze** sa Grand Central Dispatch (GCD) okvirom koji je originalno napisan u C-u.\
Biblioteka **`libswiftDispatch`** omota C GCD API-je u interfejs koji je prijateljski prema Swift-u, čineći ga lakšim i intuitivnijim za rad sa GCD-om za Swift programere.

* **`DispatchQueue.global().sync{ ... }`**
* **`DispatchQueue.global().async{ ... }`**
* **`let onceToken = DispatchOnce(); onceToken.perform { ... }`**
* **`async await`**
* **`var (data, response) = await URLSession.shared.data(from: URL(string: "https://api.example.com/getData"))`**

**Primer koda**:

```swift
import Foundation

// Define a closure (the Swift equivalent of a block)
let backgroundTask: () -> Void = {
for i in 0..<10 {
print("Background task \(i)")
sleep(1)  // Simulate a long-running task
}
}

// Entry point
autoreleasepool {
// Create a dispatch queue
let backgroundQueue = DispatchQueue(label: "com.example.backgroundQueue")

// Submit the closure to the queue for asynchronous execution
backgroundQueue.async(execute: backgroundTask)

// Continue with other work on the main queue
for i in 0..<10 {
print("Main task \(i)")
sleep(1)  // Simulate a long-running task
}
}
```

## Frida

Sledeći Frida skript može se koristiti za **hukovanje u nekoliko `dispatch`** funkcija i izvlačenje imena reda, tragova izvršavanja i bloka: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)

```bash
frida -U <prog_name> -l libdispatch.js

dispatch_sync
Calling queue: com.apple.UIKit._UIReusePool.reuseSetAccess
Callback function: 0x19e3a6488 UIKitCore!__26-[_UIReusePool addObject:]_block_invoke
Backtrace:
0x19e3a6460 UIKitCore!-[_UIReusePool addObject:]
0x19e3a5db8 UIKitCore!-[UIGraphicsRenderer _enqueueContextForReuse:]
0x19e3a57fc UIKitCore!+[UIGraphicsRenderer _destroyCGContext:withRenderer:]
[...]
```

## Ghidra

Trenutno Ghidra ne razume strukturu **`dispatch_block_t`** ObjectiveC-a, niti **`swift_dispatch_block`**.

Dakle, ako želite da ih razume, jednostavno ih možete **deklarisati**:

<figure><img src="../../.gitbook/assets/image (688).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (690).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (691).png" alt="" width="563"><figcaption></figcaption></figure>

Zatim, pronađite mesto u kodu gde se **koriste**:

{% hint style="success" %}
Zabeležite sve reference na "block" kako biste shvatili kako možete otkriti da se struktura koristi.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (692).png" alt="" width="563"><figcaption></figcaption></figure>

Desni klik na promenljivu -> Promeni tip promenljive i u ovom slučaju izaberite **`swift_dispatch_block`**:

<figure><img src="../../.gitbook/assets/image (693).png" alt="" width="563"><figcaption></figcaption></figure>

Ghidra će automatski prepraviti sve:

<figure><img src="../../.gitbook/assets/image (694).png" alt="" width="563"><figcaption></figcaption></figure>

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
