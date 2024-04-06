# Mount Namespace

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

Mount namespace je funkcionalnost Linux kernela koja omogućava izolaciju tačaka montiranja fajl sistema koje vidljive su grupi procesa. Svaki mount namespace ima svoj set tačaka montiranja fajl sistema, i **promene u tačkama montiranja u jednom namespace-u ne utiču na druge namespace-ove**. Ovo znači da procesi koji se izvršavaju u različitim mount namespace-ovima mogu imati različite poglede na hijerarhiju fajl sistema.

Mount namespace-ovi su posebno korisni u kontejnerizaciji, gde svaki kontejner treba da ima svoj fajl sistem i konfiguraciju, izolovane od drugih kontejnera i host sistema.

### Kako radi:

1. Kada se kreira novi mount namespace, inicijalizuje se sa **kopijom tačaka montiranja iz roditeljskog namespace-a**. Ovo znači da, pri kreiranju, novi namespace deli isti pogled na fajl sistem kao i njegov roditelj. Međutim, bilo kakve naknadne promene u tačkama montiranja unutar namespace-a neće uticati na roditelja ili druge namespace-ove.
2. Kada proces modifikuje tačku montiranja unutar svog namespace-a, kao što je montiranje ili demontiranje fajl sistema, **promena je lokalna za taj namespace** i ne utiče na druge namespace-ove. Ovo omogućava svakom namespace-u da ima svoju nezavisnu hijerarhiju fajl sistema.
3. Procesi mogu da se premeštaju između namespace-ova koristeći `setns()` sistemski poziv, ili da kreiraju nove namespace-ove koristeći `unshare()` ili `clone()` sistemski pozive sa `CLONE_NEWNS` zastavicom. Kada proces pređe u novi namespace ili ga kreira, počeće da koristi tačke montiranja povezane sa tim namespace-om.
4. **File deskriptori i inodi se dele između namespace-ova**, što znači da ako proces u jednom namespace-u ima otvoren file deskriptor koji pokazuje na fajl, može **proslediti taj file deskriptor** procesu u drugom namespace-u, i **oba procesa će pristupiti istom fajlu**. Međutim, putanja do fajla može biti različita u oba namespace-a zbog razlika u tačkama montiranja.

## Lab:

### Kreiranje različitih Namespace-ova

#### CLI

```bash
sudo unshare -m [--mount-proc] /bin/bash
```

Montiranjem nove instance `/proc` fajl sistema, korišćenjem parametra `--mount-proc`, obezbeđujete da nova mount namespace ima **tačan i izolovan prikaz informacija o procesima specifičnim za tu namespace**.

<details>

<summary>Greška: bash: fork: Ne može se alocirati memorija</summary>

Kada se `unshare` izvršava bez opcije `-f`, javlja se greška zbog načina na koji Linux obrađuje nove PID (Process ID) namespace-ove. Ključni detalji i rešenje su opisani u nastavku:

1. **Objašnjenje problema**:

* Linux kernel omogućava procesu da kreira nove namespace-ove koristeći `unshare` sistemski poziv. Međutim, proces koji pokreće kreiranje novog PID namespace-a (nazvan "unshare" proces) ne ulazi u novi namespace; samo njegovi dečiji procesi to čine.
* Pokretanje `%unshare -p /bin/bash%` pokreće `/bin/bash` u istom procesu kao `unshare`. Kao rezultat, `/bin/bash` i njegovi dečiji procesi su u originalnom PID namespace-u.
* Prvi dečiji proces `/bin/bash` u novom namespace-u postaje PID 1. Kada ovaj proces završi, pokreće se čišćenje namespace-a ako nema drugih procesa, jer PID 1 ima posebnu ulogu usvajanja siročadi. Linux kernel će tada onemogućiti alokaciju PID-ova u tom namespace-u.

2. **Posledica**:

* Izlazak PID 1 iz novog namespace-a dovodi do čišćenja `PIDNS_HASH_ADDING` zastavice. To rezultira neuspehom funkcije `alloc_pid` pri alociranju novog PID-a prilikom kreiranja novog procesa, što dovodi do greške "Ne može se alocirati memorija".

3. **Rešenje**:

* Problem se može rešiti korišćenjem opcije `-f` sa `unshare`. Ova opcija čini da `unshare` fork-uje novi proces nakon kreiranja novog PID namespace-a.
* Izvršavanje `%unshare -fp /bin/bash%` osigurava da sam `unshare` postane PID 1 u novom namespace-u. `/bin/bash` i njegovi dečiji procesi su tada sigurno smešteni unutar ovog novog namespace-a, sprečavajući prevremeni izlazak PID 1 i omogućavajući normalnu alokaciju PID-ova.

Obezbeđivanjem da `unshare` radi sa opcijom `-f`, novi PID namespace se pravilno održava, omogućavajući `/bin/bash` i njegovim podprocesima da rade bez greške alokacije memorije.

</details>

#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

### Proverite u kojem se namespace-u nalazi vaš proces

Da biste proverili u kojem se namespace-u nalazi vaš proces, možete koristiti sledeću komandu:

```bash
ls -l /proc/$$/ns
```

Ova komanda će vam prikazati simboličke veze koje predstavljaju različite namespace-ove u kojima se vaš proces nalazi.

```bash
ls -l /proc/self/ns/mnt
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/mnt -> 'mnt:[4026531841]'
```

### Pronađite sve Mount namespace-ove

{% code overflow="wrap" %}
```
```
{% endcode %}

```bash
sudo find /proc -maxdepth 3 -type l -name mnt -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name mnt -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```

```
```

\`\`\`bash nsenter -m TARGET\_PID --pid /bin/bash \`\`\` Takođe, možete \*\*ući u drugi procesni namespace samo ako ste root\*\*. I \*\*ne možete\*\* \*\*ući\*\* u drugi namespace \*\*bez deskriptora\*\* koji na njega ukazuje (poput \`/proc/self/ns/mnt\`).

Zato što su novi mount-ovi dostupni samo unutar namespace-a, moguće je da namespace sadrži osetljive informacije koje su dostupne samo iz njega.

### Montiraj nešto

```bash
# Generate new mount ns
unshare -m /bin/bash
mkdir /tmp/mount_ns_example
mount -t tmpfs tmpfs /tmp/mount_ns_example
mount | grep tmpfs # "tmpfs on /tmp/mount_ns_example"
echo test > /tmp/mount_ns_example/test
ls /tmp/mount_ns_example/test # Exists

# From the host
mount | grep tmpfs # Cannot see "tmpfs on /tmp/mount_ns_example"
ls /tmp/mount_ns_example/test # Doesn't exist
```

## Reference

* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
