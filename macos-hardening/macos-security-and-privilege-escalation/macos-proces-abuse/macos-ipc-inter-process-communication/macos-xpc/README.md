# macOS XPC

## macOS XPC

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### Osnovne informacije

XPC, što je skraćenica za XNU (jezgro koje koristi macOS) inter-procesnu komunikaciju, je okvir za **komunikaciju između procesa** na macOS-u i iOS-u. XPC pruža mehanizam za izvršavanje **bezbednih, asinhronih poziva metoda između različitih procesa** na sistemu. To je deo Apple-ovog sigurnosnog paradigme, koji omogućava **kreiranje aplikacija sa odvojenim privilegijama** gde svaki **komponent** radi sa **samo dozvolama koje su mu potrebne** da obavi svoj posao, čime se ograničava potencijalna šteta od kompromitovanog procesa.

XPC koristi oblik Inter-Procesne Komunikacije (IPC), koji je skup metoda za različite programe koji se izvršavaju na istom sistemu kako bi slali podatke napred i nazad.

Glavne prednosti XPC-a uključuju:

1. **Bezbednost**: Razdvajanjem posla u različite procese, svaki proces može dobiti samo dozvole koje su mu potrebne. To znači da čak i ako je proces kompromitovan, ima ograničene mogućnosti da nanese štetu.
2. **Stabilnost**: XPC pomaže izolaciju rušenja na komponentu gde se događaju. Ako se proces sruši, može se ponovo pokrenuti bez uticaja na ostatak sistema.
3. **Performanse**: XPC omogućava jednostavnu konkurentnost, jer različiti zadaci mogu se izvršavati istovremeno u različitim procesima.

Jedini **nedostatak** je da **razdvajanje aplikacije u nekoliko procesa** koji komuniciraju putem XPC-a je **manje efikasno**. Ali u današnjim sistemima to je gotovo neprimetno, a prednosti su veće.

### XPC servisi specifični za aplikaciju

XPC komponente aplikacije su **unutar same aplikacije**. Na primer, u Safari-ju ih možete pronaći u **`/Applications/Safari.app/Contents/XPCServices`**. Imaju ekstenziju **`.xpc`** (kao **`com.apple.Safari.SandboxBroker.xpc`**) i **takođe su paketi** sa glavnim binarnim fajlom unutar njega: `/Applications/Safari.app/Contents/XPCServices/com.apple.Safari.SandboxBroker.xpc/Contents/MacOS/com.apple.Safari.SandboxBroker` i `Info.plist: /Applications/Safari.app/Contents/XPCServices/com.apple.Safari.SandboxBroker.xpc/Contents/Info.plist`

Kao što možda mislite, **XPC komponenta će imati različite privilegije i dozvole** od drugih XPC komponenti ili glavnog binarnog fajla aplikacije. OSIM ako je XPC servis konfigurisan sa [**JoinExistingSession**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/xpcservice/joinexistingsession) postavljenim na "True" u svom **Info.plist** fajlu. U tom slučaju, XPC servis će se izvršavati u **istoj sigurnosnoj sesiji kao aplikacija** koja ga je pozvala.

XPC servisi se **pokreću** pomoću **launchd** kada su potrebni i **zaustavljaju** se kada su sve zadatke **završili** kako bi se oslobodili sistemski resursi. **XPC komponente specifične za aplikaciju mogu koristiti samo aplikacija**, čime se smanjuje rizik od potencijalnih ranjivosti.

### XPC servisi na nivou sistema

XPC servisi na nivou sistema su dostupni svim korisnicima. Ovi servisi, bilo da su u pitanju launchd ili Mach-tip, moraju biti **definisani u plist** fajlovima koji se nalaze u određenim direktorijumima kao što su **`/System/Library/LaunchDaemons`**, **`/Library/LaunchDaemons`**, **`/System/Library/LaunchAgents`**, ili **`/Library/LaunchAgents`**.

Ovi plist fajlovi će imati ključ pod nazivom **`MachServices`** sa imenom servisa, i ključ pod nazivom **`Program`** sa putanjom do binarnog fajla:

```xml
cat /Library/LaunchDaemons/com.jamf.management.daemon.plist

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Program</key>
<string>/Library/Application Support/JAMF/Jamf.app/Contents/MacOS/JamfDaemon.app/Contents/MacOS/JamfDaemon</string>
<key>AbandonProcessGroup</key>
<true/>
<key>KeepAlive</key>
<true/>
<key>Label</key>
<string>com.jamf.management.daemon</string>
<key>MachServices</key>
<dict>
<key>com.jamf.management.daemon.aad</key>
<true/>
<key>com.jamf.management.daemon.agent</key>
<true/>
<key>com.jamf.management.daemon.binary</key>
<true/>
<key>com.jamf.management.daemon.selfservice</key>
<true/>
<key>com.jamf.management.daemon.service</key>
<true/>
</dict>
<key>RunAtLoad</key>
<true/>
</dict>
</plist>
```

Oni u **`LaunchDameons`** direktorijumu se pokreću kao root. Dakle, ako neprivilegovani proces može da komunicira sa jednim od njih, mogao bi da dobije privilegije.

### XPC Event poruke

Aplikacije mogu **pretplatiti** na različite događajne **poruke**, omogućavajući im da se **iniciraju po potrebi** kada se takvi događaji dese. **Postavljanje** ovih usluga se vrši u **launchd plist fajlovima**, smeštenim u **istim direktorijumima kao i prethodni**, i sadrže dodatni ključ **`LaunchEvent`**.

#### Provera povezanog XPC procesa

Kada proces pokuša da pozove metod putem XPC konekcije, **XPC servis treba da proveri da li je taj proces dozvoljen da se poveže**. Evo uobičajenih načina za proveru i uobičajenih zamki:

{% content-ref url="macos-xpc-connecting-process-check/" %}
[macos-xpc-connecting-process-check](macos-xpc-connecting-process-check/)
{% endcontent-ref %}

### XPC Autorizacija

Apple takođe omogućava aplikacijama da **konfigurišu određena prava i način njihovog dobijanja**, tako da ako pozivajući proces ima ta prava, biće **dozvoljeno pozivanje metoda** iz XPC servisa:

{% content-ref url="macos-xpc-authorization.md" %}
[macos-xpc-authorization.md](macos-xpc-authorization.md)
{% endcontent-ref %}

### XPC Sniffer

Da biste prisluškivali XPC poruke, možete koristiti [**xpcspy**](https://github.com/hot3eed/xpcspy) koji koristi **Frida**.

```bash
# Install
pip3 install xpcspy
pip3 install xpcspy --no-deps # To not make xpcspy install Frida 15 and downgrade your Frida installation

# Start sniffing
xpcspy -U -r -W <bundle-id>
## Using filters (i: for input, o: for output)
xpcspy -U <prog-name> -t 'i:com.apple.*' -t 'o:com.apple.*' -r
```

### XPC Komunikacija C Primer koda

{% tabs %}
{% tab title="undefined" %}
```c
// gcc xpc_server.c -o xpc_server

#include <xpc/xpc.h>

static void handle_event(xpc_object_t event) {
if (xpc_get_type(event) == XPC_TYPE_DICTIONARY) {
// Print received message
const char* received_message = xpc_dictionary_get_string(event, "message");
printf("Received message: %s\n", received_message);

// Create a response dictionary
xpc_object_t response = xpc_dictionary_create(NULL, NULL, 0);
xpc_dictionary_set_string(response, "received", "received");

// Send response
xpc_connection_t remote = xpc_dictionary_get_remote_connection(event);
xpc_connection_send_message(remote, response);

// Clean up
xpc_release(response);
}
}

static void handle_connection(xpc_connection_t connection) {
xpc_connection_set_event_handler(connection, ^(xpc_object_t event) {
handle_event(event);
});
xpc_connection_resume(connection);
}

int main(int argc, const char *argv[]) {
xpc_connection_t service = xpc_connection_create_mach_service("xyz.hacktricks.service",
dispatch_get_main_queue(),
XPC_CONNECTION_MACH_SERVICE_LISTENER);
if (!service) {
fprintf(stderr, "Failed to create service.\n");
exit(EXIT_FAILURE);
}

xpc_connection_set_event_handler(service, ^(xpc_object_t event) {
xpc_type_t type = xpc_get_type(event);
if (type == XPC_TYPE_CONNECTION) {
handle_connection(event);
}
});

xpc_connection_resume(service);
dispatch_main();

return 0;
}
```
{% endtab %}

{% tab title="undefined" %}
```c
// gcc xpc_client.c -o xpc_client

#include <xpc/xpc.h>

int main(int argc, const char *argv[]) {
xpc_connection_t connection = xpc_connection_create_mach_service("xyz.hacktricks.service", NULL, XPC_CONNECTION_MACH_SERVICE_PRIVILEGED);

xpc_connection_set_event_handler(connection, ^(xpc_object_t event) {
if (xpc_get_type(event) == XPC_TYPE_DICTIONARY) {
// Print received message
const char* received_message = xpc_dictionary_get_string(event, "received");
printf("Received message: %s\n", received_message);
}
});

xpc_connection_resume(connection);

xpc_object_t message = xpc_dictionary_create(NULL, NULL, 0);
xpc_dictionary_set_string(message, "message", "Hello, Server!");

xpc_connection_send_message(connection, message);

dispatch_main();

return 0;
}
```
{% endtab %}

{% tab title="xyz.hacktricks.service.plist" %}
xyz.hacktricks.service.plist je datoteka koja se koristi za konfigurisanje XPC servisa na macOS operativnom sistemu. XPC (Cross-Process Communication) je mehanizam koji omogućava komunikaciju između različitih procesa na macOS-u. Ova datoteka definiše parametre i postavke za XPC servis, kao što su putanja do izvršne datoteke, argumenti komandne linije i okruženje. Kada se pokrene XPC servis, macOS koristi ovu datoteku za konfigurisanje servisa i uspostavljanje komunikacije sa drugim procesima.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>Label</key>
<string>xyz.hacktricks.service</string>
<key>MachServices</key>
<dict>
<key>xyz.hacktricks.service</key>
<true/>
</dict>
<key>Program</key>
<string>/tmp/xpc_server</string>
<key>ProgramArguments</key>
<array>
<string>/tmp/xpc_server</string>
</array>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

```bash
# Compile the server & client
gcc xpc_server.c -o xpc_server
gcc xpc_client.c -o xpc_client

# Save server on it's location
cp xpc_server /tmp

# Load daemon
sudo cp xyz.hacktricks.service.plist /Library/LaunchDaemons
sudo launchctl load /Library/LaunchDaemons/xyz.hacktricks.service.plist

# Call client
./xpc_client

# Clean
sudo launchctl unload /Library/LaunchDaemons/xyz.hacktricks.service.plist
sudo rm /Library/LaunchDaemons/xyz.hacktricks.service.plist /tmp/xpc_server
```

### Primer koda za XPC komunikaciju u Objective-C-u

{% tabs %}
{% tab title="undefined" %}
```objectivec
// gcc -framework Foundation oc_xpc_server.m -o oc_xpc_server
#include <Foundation/Foundation.h>

@protocol MyXPCProtocol
- (void)sayHello:(NSString *)some_string withReply:(void (^)(NSString *))reply;
@end

@interface MyXPCObject : NSObject <MyXPCProtocol>
@end


@implementation MyXPCObject
- (void)sayHello:(NSString *)some_string withReply:(void (^)(NSString *))reply {
NSLog(@"Received message: %@", some_string);
NSString *response = @"Received";
reply(response);
}
@end

@interface MyDelegate : NSObject <NSXPCListenerDelegate>
@end


@implementation MyDelegate

- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
newConnection.exportedInterface = [NSXPCInterface interfaceWithProtocol:@protocol(MyXPCProtocol)];

MyXPCObject *my_object = [MyXPCObject new];

newConnection.exportedObject = my_object;

[newConnection resume];
return YES;
}
@end

int main(void) {

NSXPCListener *listener = [[NSXPCListener alloc] initWithMachServiceName:@"xyz.hacktricks.svcoc"];

id <NSXPCListenerDelegate> delegate = [MyDelegate new];
listener.delegate = delegate;
[listener resume];

sleep(10); // Fake something is done and then it ends
}
```
{% endtab %}

{% tab title="undefined" %}
```objectivec
// gcc -framework Foundation oc_xpc_client.m -o oc_xpc_client
#include <Foundation/Foundation.h>

@protocol MyXPCProtocol
- (void)sayHello:(NSString *)some_string withReply:(void (^)(NSString *))reply;
@end

int main(void) {
NSXPCConnection *connection = [[NSXPCConnection alloc] initWithMachServiceName:@"xyz.hacktricks.svcoc" options:NSXPCConnectionPrivileged];
connection.remoteObjectInterface = [NSXPCInterface interfaceWithProtocol:@protocol(MyXPCProtocol)];
[connection resume];

[[connection remoteObjectProxy] sayHello:@"Hello, Server!" withReply:^(NSString *response) {
NSLog(@"Received response: %@", response);
}];

[[NSRunLoop currentRunLoop] run];

return 0;
}
```
{% endtab %}

{% tab title="xyz.hacktricks.svcoc.plist" %}
xyz.hacktricks.svcoc.plist je konfiguracioni fajl za XPC servis na macOS-u. XPC (XPC Services) je mehanizam za interprocesnu komunikaciju (IPC) koji omogućava komunikaciju između različitih procesa na macOS-u. Ovaj fajl definiše kako će se XPC servis pokrenuti i konfigurisati.

Da biste iskoristili XPC servis, možete izmeniti ovaj fajl kako biste promenili način na koji se servis pokreće ili konfiguriše. Na primer, možete promeniti putanju do izvršnog fajla servisa ili dodati dodatne argumente za pokretanje.

Važno je napomenuti da izmena ovog fajla može dovesti do neispravnog rada XPC servisa ili čak do sigurnosnih propusta. Stoga, pre nego što izmenite ovaj fajl, preporučuje se da pažljivo proučite dokumentaciju i razumete kako XPC servisi funkcionišu na macOS-u. Takođe, preporučuje se da napravite rezervnu kopiju originalnog fajla pre izmene kako biste mogli da se vratite na prethodno stanje ako nešto pođe po zlu.

Ukratko, xyz.hacktricks.svcoc.plist je konfiguracioni fajl za XPC servis na macOS-u koji definiše način pokretanja i konfiguracije servisa. Izmenom ovog fajla možete prilagoditi ponašanje XPC servisa, ali budite oprezni da ne izazovete neispravan rad ili sigurnosne propuste.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>Label</key>
<string>xyz.hacktricks.svcoc</string>
<key>MachServices</key>
<dict>
<key>xyz.hacktricks.svcoc</key>
<true/>
</dict>
<key>Program</key>
<string>/tmp/oc_xpc_server</string>
<key>ProgramArguments</key>
<array>
<string>/tmp/oc_xpc_server</string>
</array>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

\`\`\`bash # Compile the server & client gcc -framework Foundation oc\_xpc\_server.m -o oc\_xpc\_server gcc -framework Foundation oc\_xpc\_client.m -o oc\_xpc\_client

## Save server on it's location

cp oc\_xpc\_server /tmp

## Load daemon

sudo cp xyz.hacktricks.svcoc.plist /Library/LaunchDaemons sudo launchctl load /Library/LaunchDaemons/xyz.hacktricks.svcoc.plist

## Call client

./oc\_xpc\_client

## Clean

sudo launchctl unload /Library/LaunchDaemons/xyz.hacktricks.svcoc.plist sudo rm /Library/LaunchDaemons/xyz.hacktricks.svcoc.plist /tmp/oc\_xpc\_server

````
## Klijent unutar Dylb koda

### Opis

Dylb je biblioteka za dinamičko učitavanje koda na macOS operativnom sistemu. Ova biblioteka omogućava izvršavanje koda unutar drugih procesa. Kada se koristi u kombinaciji sa XPC (Inter-procesna komunikacija) mehanizmom, Dylb može biti iskorišćen za postizanje privilegija eskalacije i zloupotrebu procesa na macOS sistemu.

### Klijent unutar Dylb koda

Klijent unutar Dylb koda je tehnika koja omogućava izvršavanje koda unutar ciljnog procesa putem Dylb biblioteke. Ova tehnika se koristi za komunikaciju sa XPC servisima unutar ciljnog procesa i izvršavanje određenih funkcionalnosti.

Da bi se koristila ova tehnika, potrebno je prvo učitati Dylb biblioteku unutar ciljnog procesa. Zatim se može uspostaviti komunikacija sa XPC servisima i izvršiti odgovarajuće funkcije.

Ova tehnika može biti korisna u situacijama kada je potrebno izvršiti određene zadatke unutar ciljnog procesa, kao što je prikupljanje informacija ili izvršavanje određenih komandi.

### Primer koda

```c
#include <dlfcn.h>
#include <stdio.h>

int main() {
    void* handle = dlopen("/path/to/dylb/library.dylib", RTLD_LAZY);
    if (handle == NULL) {
        printf("Failed to load Dylb library\n");
        return 1;
    }

    // Učitavanje funkcije iz Dylb biblioteke
    void (*executeCode)(void) = dlsym(handle, "executeCode");
    if (executeCode == NULL) {
        printf("Failed to find executeCode function\n");
        dlclose(handle);
        return 1;
    }

    // Izvršavanje funkcije unutar ciljnog procesa
    executeCode();

    dlclose(handle);
    return 0;
}
````

#### Zaključak

Korišćenje klijenta unutar Dylb koda omogućava izvršavanje koda unutar ciljnog procesa putem Dylb biblioteke. Ova tehnika može biti korisna za postizanje privilegija eskalacije i zloupotrebu procesa na macOS sistemu.

```objectivec
// gcc -dynamiclib -framework Foundation oc_xpc_client.m -o oc_xpc_client.dylib
// gcc injection example:
// DYLD_INSERT_LIBRARIES=oc_xpc_client.dylib /path/to/vuln/bin

#import <Foundation/Foundation.h>

@protocol MyXPCProtocol
- (void)sayHello:(NSString *)some_string withReply:(void (^)(NSString *))reply;
@end

__attribute__((constructor))
static void customConstructor(int argc, const char **argv)
{
NSString*  _serviceName = @"xyz.hacktricks.svcoc";

NSXPCConnection* _agentConnection = [[NSXPCConnection alloc] initWithMachServiceName:_serviceName options:4096];

[_agentConnection setRemoteObjectInterface:[NSXPCInterface interfaceWithProtocol:@protocol(MyXPCProtocol)]];

[_agentConnection resume];

[[_agentConnection remoteObjectProxyWithErrorHandler:^(NSError* error) {
(void)error;
NSLog(@"Connection Failure");
}] sayHello:@"Hello, Server!" withReply:^(NSString *response) {
NSLog(@"Received response: %@", response);
}    ];
NSLog(@"Done!");

return;
}
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
