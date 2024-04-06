# IPC Namespace

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovne informacije

IPC (Inter-Process Communication) namespace je funkcionalnost Linux kernela koja pruža **izolaciju** System V IPC objekata, kao što su redovi poruka, segmenti deljene memorije i semafori. Ova izolacija obezbeđuje da procesi u **različitim IPC namespace-ima ne mogu direktno pristupati ili menjati IPC objekte drugih namespace-ova**, pružajući dodatni sloj sigurnosti i privatnosti između grupa procesa.

### Kako funkcioniše:

1. Kada se kreira novi IPC namespace, on počinje sa **potpuno izolovanim skupom System V IPC objekata**. Ovo znači da procesi koji se izvršavaju u novom IPC namespace-u ne mogu pristupiti ili ometati IPC objekte u drugim namespace-ovima ili na host sistemu po default-u.
2. IPC objekti kreirani unutar namespace-a su vidljivi i **pristupačni samo procesima unutar tog namespace-a**. Svaki IPC objekat je identifikovan jedinstvenim ključem unutar svog namespace-a. Iako ključ može biti identičan u različitim namespace-ovima, sami objekti su izolovani i ne mogu se pristupiti preko namespace-ova.
3. Procesi mogu da se premeštaju između namespace-ova koristeći `setns()` sistemski poziv ili kreiraju nove namespace-ove koristeći `unshare()` ili `clone()` sistemski pozivi sa `CLONE_NEWIPC` zastavicom. Kada proces pređe u novi namespace ili ga kreira, počeće da koristi IPC objekte povezane sa tim namespace-om.

## Laboratorija:

### Kreiranje različitih Namespace-ova

#### CLI

```bash
sudo unshare -i [--mount-proc] /bin/bash
```

Montiranjem nove instance `/proc` fajl sistema, korišćenjem parametra `--mount-proc`, obezbeđujete da nova montirana namespace ima **tačan i izolovan prikaz informacija o procesima specifičnim za tu namespace**.

<details>

<summary>Greška: bash: fork: Ne može se alocirati memorija</summary>

Kada se `unshare` izvršava bez opcije `-f`, javlja se greška zbog načina na koji Linux rukuje novim PID (Process ID) namespace-om. Ključni detalji i rešenje su opisani u nastavku:

1. **Objašnjenje problema**:

* Linux kernel omogućava procesu da kreira nove namespace-ove koristeći `unshare` sistemski poziv. Međutim, proces koji pokreće kreiranje novog PID namespace-a (nazvan "unshare" proces) ne ulazi u novi namespace; samo njegovi podprocesi to čine.
* Pokretanje `%unshare -p /bin/bash%` pokreće `/bin/bash` u istom procesu kao `unshare`. Kao rezultat, `/bin/bash` i njegovi podprocesi su u originalnom PID namespace-u.
* Prvi podproces `/bin/bash` u novom namespace-u postaje PID 1. Kada ovaj proces završi, pokreće se čišćenje namespace-a ako nema drugih procesa, jer PID 1 ima posebnu ulogu usvajanja siročadi. Linux kernel tada onemogućava alokaciju PID-a u tom namespace-u.

2. **Posledica**:

* Izlazak PID 1 iz novog namespace-a dovodi do čišćenja `PIDNS_HASH_ADDING` zastavice. To rezultira neuspehom funkcije `alloc_pid` pri alociranju novog PID-a prilikom kreiranja novog procesa, što dovodi do greške "Ne može se alocirati memorija".

3. **Rešenje**:

* Problem se može rešiti korišćenjem opcije `-f` sa `unshare`. Ova opcija čini da `unshare` fork-uje novi proces nakon kreiranja novog PID namespace-a.
* Izvršavanje `%unshare -fp /bin/bash%` osigurava da sam `unshare` komanda postane PID 1 u novom namespace-u. `/bin/bash` i njegovi podprocesi su tada sigurno smešteni unutar ovog novog namespace-a, sprečavajući prevremeni izlazak PID 1 i omogućavajući normalnu alokaciju PID-a.

Obezbeđivanjem da `unshare` radi sa opcijom `-f`, novi PID namespace se pravilno održava, omogućavajući `/bin/bash` i njegovim podprocesima da rade bez greške alociranja memorije.

</details>

#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

### Proverite u kojem se namespace-u nalazi vaš proces

Da biste proverili u kojem se namespace-u nalazi vaš proces, možete koristiti sledeću komandu:

```bash
ls -l /proc/$$/ns/ipc
```

Ova komanda će vam prikazati simboličku vezu koja pokazuje na IPC namespace u kojem se trenutno nalazi vaš proces.

```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```

### Pronađite sve IPC namespace-ove

{% code overflow="wrap" %}
```
```
{% endcode %}

```bash
sudo find /proc -maxdepth 3 -type l -name ipc -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name ipc -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```

```
```

\`\`\`bash nsenter -i TARGET\_PID --pid /bin/bash \`\`\` Takođe, možete \*\*ući u drugi procesni namespace samo ako ste root\*\*. I \*\*ne možete\*\* \*\*ući\*\* u drugi namespace \*\*bez deskriptora\*\* koji na njega ukazuje (poput \`/proc/self/ns/net\`).

### Kreiranje IPC objekta

```bash
# Container
sudo unshare -i /bin/bash
ipcmk -M 100
Shared memory id: 0
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x2fba9021 0          root       644        100        0

# From the host
ipcs -m # Nothing is seen
```

## Reference

* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
