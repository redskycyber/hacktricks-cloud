# UTS Namespace

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

UTS (UNIX Time-Sharing System) namespace je funkcionalnost Linux kernela koja pruža **izolaciju dva sistema identifikatora**: **hostname-a** i **NIS** (Network Information Service) domenskog imena. Ova izolacija omogućava svakom UTS namespace-u da ima svoj **nezavisni hostname i NIS domensko ime**, što je posebno korisno u scenarijima kontejnerizacije gde svaki kontejner treba da se pojavi kao zaseban sistem sa svojim sopstvenim hostname-om.

### Kako radi:

1. Kada se kreira novi UTS namespace, on počinje sa **kopijom hostname-a i NIS domenskog imena iz roditeljskog namespace-a**. Ovo znači da, pri kreiranju, novi namespace **deli iste identifikatore kao i njegov roditelj**. Međutim, bilo kakve naknadne promene hostname-a ili NIS domenskog imena unutar namespace-a neće uticati na druge namespace-ove.
2. Procesi unutar UTS namespace-a **mogu promeniti hostname i NIS domensko ime** koristeći sistemski poziv `sethostname()` i `setdomainname()`, redom. Ove promene su lokalne za namespace i ne utiču na druge namespace-ove ili host sistem.
3. Procesi mogu prelaziti između namespace-ova koristeći sistemski poziv `setns()` ili kreirati nove namespace-ove koristeći sistemski poziv `unshare()` ili `clone()` sa `CLONE_NEWUTS` flag-om. Kada proces pređe u novi namespace ili ga kreira, počeće da koristi hostname i NIS domensko ime povezano sa tim namespace-om.

## Laboratorija:

### Kreiranje različitih Namespace-ova

#### CLI

```bash
sudo unshare -u [--mount-proc] /bin/bash
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
cat /proc/$$/ns/uts
```

Ova komanda će vam prikazati putanju do fajla koji predstavlja UTS namespace vašeg procesa.

```bash
ls -l /proc/self/ns/uts
lrwxrwxrwx 1 root root 0 Apr  4 20:49 /proc/self/ns/uts -> 'uts:[4026531838]'
```

### Pronađite sve UTS namespace-ove

{% code overflow="wrap" %}
```
```
{% endcode %}

```bash
sudo find /proc -maxdepth 3 -type l -name uts -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name uts -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```

```
```

\`\`\`bash nsenter -u TARGET\_PID --pid /bin/bash \`\`\` Takođe, možete \*\*ući u drugi proces namespace samo ako ste root\*\*. I \*\*ne možete\*\* \*\*ući\*\* u drugi namespace \*\*bez deskriptora\*\* koji na njega ukazuje (poput \`/proc/self/ns/uts\`).

### Promena imena hosta

```bash
unshare -u /bin/bash
hostname newhostname # Hostname won't be changed inside the host UTS ns
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
