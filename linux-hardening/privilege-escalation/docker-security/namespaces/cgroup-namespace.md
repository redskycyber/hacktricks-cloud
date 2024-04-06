# CGroup Namespace

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

CGroup namespace je funkcionalnost Linux kernela koja pruža **izolaciju hijerarhija cgroup-ova za procese koji se izvršavaju unutar namespace-a**. Cgroup-ovi, skraćeno za **control groups**, su funkcionalnost kernela koja omogućava organizovanje procesa u hijerarhijske grupe radi upravljanja i sprovođenja **ograničenja na sistemskim resursima** kao što su CPU, memorija i I/O.

Iako cgroup namespace-i nisu poseban tip namespace-a kao što su PID, mount, network, itd., oni su povezani sa konceptom izolacije namespace-a. **Cgroup namespace-i virtualizuju prikaz hijerarhije cgroup-ova**, tako da procesi koji se izvršavaju unutar cgroup namespace-a imaju drugačiji prikaz hijerarhije u odnosu na procese koji se izvršavaju na hostu ili drugim namespace-ima.

### Kako funkcioniše:

1. Kada se kreira novi cgroup namespace, **on počinje sa prikazom hijerarhije cgroup-ova zasnovanom na cgroup-u procesa koji ga kreira**. To znači da će procesi koji se izvršavaju u novom cgroup namespace-u videti samo podskup celokupne hijerarhije cgroup-ova, ograničen na podstablo cgroup-a koje ima korenski čvor u cgroup-u procesa koji ga kreira.
2. Procesi unutar cgroup namespace-a će **videti svoj sopstveni cgroup kao koren hijerarhije**. To znači da, iz perspektive procesa unutar namespace-a, njihov sopstveni cgroup će se prikazivati kao koren, i oni neće moći videti ili pristupiti cgroup-ovima van svog sopstvenog podstabla.
3. Cgroup namespace-i ne pružaju direktnu izolaciju resursa; **oni samo pružaju izolaciju prikaza hijerarhije cgroup-ova**. **Kontrola i izolacija resursa se i dalje sprovode putem podsistema cgroup-ova** (npr. cpu, memorija, itd.) samih.

Za više informacija o CGroup-ovima pogledajte:

{% content-ref url="../cgroups.md" %}
[cgroups.md](../cgroups.md)
{% endcontent-ref %}

## Lab:

### Kreiranje različitih Namespace-ova

#### CLI

```bash
sudo unshare -C [--mount-proc] /bin/bash
```

Montiranjem nove instance `/proc` fajl sistema, korišćenjem parametra `--mount-proc`, obezbeđujete da nova mount namespace ima **tačan i izolovan prikaz informacija o procesima specifičnim za tu namespace**.

<details>

<summary>Greška: bash: fork: Ne može se alocirati memorija</summary>

Kada se `unshare` izvršava bez opcije `-f`, javlja se greška zbog načina na koji Linux rukuje novim PID (Process ID) namespace-om. Ključni detalji i rešenje su opisani u nastavku:

1. **Objašnjenje problema**:

* Linux kernel omogućava procesu da kreira nove namespace-ove koristeći `unshare` sistemski poziv. Međutim, proces koji pokreće kreiranje novog PID namespace-a (nazvan "unshare" proces) ne ulazi u novi namespace; samo njegovi podprocesi to čine.
* Pokretanje `%unshare -p /bin/bash%` pokreće `/bin/bash` u istom procesu kao `unshare`. Kao rezultat, `/bin/bash` i njegovi podprocesi su u originalnom PID namespace-u.
* Prvi podproces `/bin/bash` u novom namespace-u postaje PID 1. Kada ovaj proces završi, pokreće se čišćenje namespace-a ako nema drugih procesa, jer PID 1 ima posebnu ulogu usvajanja siročadi. Linux kernel tada onemogućava alokaciju PID-ova u tom namespace-u.

2. **Posledica**:

* Izlazak PID 1 iz novog namespace-a dovodi do čišćenja `PIDNS_HASH_ADDING` zastavice. To rezultira neuspehom funkcije `alloc_pid` pri alociranju novog PID-a prilikom kreiranja novog procesa, što dovodi do greške "Ne može se alocirati memorija".

3. **Rešenje**:

* Problem se može rešiti korišćenjem opcije `-f` sa `unshare`. Ova opcija čini da `unshare` fork-uje novi proces nakon kreiranja novog PID namespace-a.
* Izvršavanje `%unshare -fp /bin/bash%` osigurava da sam `unshare` komanda postane PID 1 u novom namespace-u. `/bin/bash` i njegovi podprocesi su tada sigurno smešteni unutar ovog novog namespace-a, sprečavajući prevremeni izlazak PID 1 i omogućavajući normalnu alokaciju PID-ova.

Obezbeđivanjem da `unshare` radi sa opcijom `-f`, novi PID namespace se pravilno održava, omogućavajući `/bin/bash` i njegovim podprocesima da rade bez greške alokacije memorije.

</details>

#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

### Proverite u kojem se namespace-u nalazi vaš proces

Da biste proverili u kojem se namespace-u nalazi vaš proces, možete koristiti sledeću komandu:

```bash
cat /proc/$$/cgroup
```

Ova komanda će vam prikazati informacije o kontrolnoj grupi (cgroup) kojoj pripada vaš proces. Ako se vaš proces nalazi u cgroup-namespace-u, videćete putanju koja počinje sa `/docker/` ili `/lxc/`. Na primer, ako vidite `/docker/1234567890abcdef`, to znači da se vaš proces nalazi u cgroup-namespace-u.

Ova informacija može biti korisna prilikom istraživanja i testiranja sigurnosti Docker kontejnera i drugih sistema koji koriste namespace-ove.

```bash
ls -l /proc/self/ns/cgroup
lrwxrwxrwx 1 root root 0 Apr  4 21:19 /proc/self/ns/cgroup -> 'cgroup:[4026531835]'
```

### Pronađite sve CGroup namespace-ove

{% code overflow="wrap" %}
```
```
{% endcode %}

```bash
sudo find /proc -maxdepth 3 -type l -name cgroup -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name cgroup -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```

```
```

\`\`\`bash nsenter -C TARGET\_PID --pid /bin/bash \`\`\` Takođe, možete \*\*ući u drugi proces namespace samo ako ste root\*\*. I \*\*ne možete\*\* \*\*ući\*\* u drugi namespace \*\*bez deskriptora\*\* koji na njega pokazuje (poput \`/proc/self/ns/cgroup\`).

## Reference

* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
