# User Namespace

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

User namespace je funkcionalnost Linux kernela koja **omogućava izolaciju korisničkih i grupnih ID mapiranja**, što omogućava svakom user namespace-u da ima svoj **skup korisničkih i grupnih ID-ova**. Ova izolacija omogućava procesima koji se izvršavaju u različitim user namespace-ima da **imaju različite privilegije i vlasništvo**, čak i ako numerički dele iste korisničke i grupne ID-ove.

User namespace-ovi su posebno korisni u kontejnerizaciji, gde svaki kontejner treba da ima svoj nezavisan skup korisničkih i grupnih ID-ova, što omogućava bolju bezbednost i izolaciju između kontejnera i host sistema.

### Kako radi:

1. Kada se kreira novi user namespace, **počinje sa praznim skupom korisničkih i grupnih ID-ova**. To znači da će bilo koji proces koji se izvršava u novom user namespace-u **inicijalno nemati privilegije izvan namespace-a**.
2. Mapiranja ID-ova mogu se uspostaviti između korisničkih i grupnih ID-ova u novom namespace-u i onih u roditeljskom (ili host) namespace-u. Ovo **omogućava procesima u novom namespace-u da imaju privilegije i vlasništvo koje odgovaraju korisničkim i grupnim ID-ovima u roditeljskom namespace-u**. Međutim, mapiranja ID-ova mogu biti ograničena na određene opsege i podskupove ID-ova, što omogućava preciznu kontrolu nad privilegijama koje se dodeljuju procesima u novom namespace-u.
3. Unutar user namespace-a, **procesi mogu imati punu root privilegiju (UID 0) za operacije unutar namespace-a**, dok istovremeno imaju ograničene privilegije izvan namespace-a. Ovo omogućava **kontejnerima da se izvršavaju sa privilegijama sličnim root-u unutar svog sopstvenog namespace-a, bez potpune root privilegije na host sistemu**.
4. Procesi mogu prelaziti između namespace-a koristeći `setns()` sistemski poziv ili kreirati nove namespace-e koristeći `unshare()` ili `clone()` sistemski pozive sa `CLONE_NEWUSER` zastavicom. Kada proces pređe u novi namespace ili ga kreira, počeće da koristi mapiranja korisničkih i grupnih ID-ova koja su povezana sa tim namespace-om.

## Lab:

### Kreiranje različitih Namespace-ova

#### CLI

```bash
sudo unshare -U [--mount-proc] /bin/bash
```

Montiranjem nove instance `/proc` fajl sistema, koristeći parametar `--mount-proc`, obezbeđujete da nova namespace montaža ima **tačan i izolovan prikaz informacija o procesima specifičnim za tu namespace**.

<details>

<summary>Greška: bash: fork: Ne može se alocirati memorija</summary>

Kada se `unshare` izvršava bez opcije `-f`, javlja se greška zbog načina na koji Linux obrađuje nove PID (Process ID) namespace-ove. Ključni detalji i rešenje su opisani u nastavku:

1. **Objašnjenje problema**:

* Linux kernel omogućava procesu da kreira nove namespace-ove koristeći `unshare` sistemski poziv. Međutim, proces koji pokreće kreiranje novog PID namespace-a (nazvan "unshare" proces) ne ulazi u novi namespace; samo njegovi podprocesi to čine.
* Pokretanje `%unshare -p /bin/bash%` pokreće `/bin/bash` u istom procesu kao `unshare`. Kao rezultat, `/bin/bash` i njegovi podprocesi su u originalnom PID namespace-u.
* Prvi podproces `/bin/bash` u novom namespace-u postaje PID 1. Kada ovaj proces završi, pokreće se čišćenje namespace-a ako nema drugih procesa, jer PID 1 ima posebnu ulogu usvajanja siročadi. Linux kernel tada onemogućava alokaciju PID-ova u tom namespace-u.

2. **Posledica**:

* Izlazak PID 1 iz novog namespace-a dovodi do čišćenja `PIDNS_HASH_ADDING` zastavice. To rezultira neuspehom funkcije `alloc_pid` pri alociranju novog PID-a prilikom kreiranja novog procesa, što dovodi do greške "Ne može se alocirati memorija".

3. **Rešenje**:

* Problem se može rešiti korišćenjem opcije `-f` sa `unshare`. Ova opcija čini da `unshare` fork-uje novi proces nakon kreiranja novog PID namespace-a.
* Izvršavanje `%unshare -fp /bin/bash%` osigurava da sam `unshare` postane PID 1 u novom namespace-u. `/bin/bash` i njegovi podprocesi su tada sigurno smešteni unutar ovog novog namespace-a, sprečavajući prevremeni izlazak PID 1 i omogućavajući normalnu alokaciju PID-ova.

Obezbeđivanjem da `unshare` radi sa opcijom `-f`, novi PID namespace se pravilno održava, omogućavajući `/bin/bash` i njegovim podprocesima da rade bez greške alociranja memorije.

</details>

#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

Da biste koristili korisnički prostor imena, Docker demon mora biti pokrenut sa **`--userns-remap=default`** (U Ubuntu 14.04, ovo se može postići izmenom `/etc/default/docker` datoteke, a zatim izvršavanjem `sudo service docker restart`).

### Proverite u kojem se prostoru imena nalazi vaš proces

```bash
ls -l /proc/self/ns/user
lrwxrwxrwx 1 root root 0 Apr  4 20:57 /proc/self/ns/user -> 'user:[4026531837]'
```

Moguće je proveriti mapu korisnika iz Docker kontejnera pomoću:

```bash
cat /proc/self/uid_map
0          0 4294967295  --> Root is root in host
0     231072      65536  --> Root is 231072 userid in host
```

Ili sa domaćina sa:

```bash
cat /proc/<pid>/uid_map
```

### Pronađite sve korisničke namespace-ove

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name user -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name user -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Uđite unutar korisničkog namespace-a

Kada se izvrši ova komanda, korisnik će biti prebačen u novi korisnički namespace. Ovo omogućava korisniku da izvršava komande sa privilegijama korisnika root unutar tog namespace-a, iako je zapravo običan korisnik na sistemu. Ovo može biti korisno za izvršavanje komandi koje zahtevaju privilegije root-a, bez potrebe za stvarnim root pristupom.

```bash
nsenter -U TARGET_PID --pid /bin/bash
```

Takođe, možete **ući u drugi procesni namespace samo ako ste root**. I **ne možete** **ući** u drugi namespace **bez deskriptora** koji na njega ukazuje (poput `/proc/self/ns/user`).

### Kreiranje novog User namespace-a (sa mapiranjima)

{% code overflow="wrap" %}
```bash
unshare -U [--map-user=<uid>|<name>] [--map-group=<gid>|<name>] [--map-root-user] [--map-current-user]
```
{% endcode %}

```bash
# Container
sudo unshare -U /bin/bash
nobody@ip-172-31-28-169:/home/ubuntu$ #Check how the user is nobody

# From the host
ps -ef | grep bash # The user inside the host is still root, not nobody
root       27756   27755  0 21:11 pts/10   00:00:00 /bin/bash
```

### Obnavljanje sposobnosti

U slučaju korisničkih namespace-ova, **kada se kreira novi korisnički namespace, proces koji ulazi u namespace dobija pun set sposobnosti unutar tog namespace-a**. Ove sposobnosti omogućavaju procesu da izvršava privilegovane operacije kao što su **montiranje** **fajl sistema**, kreiranje uređaja ili menjanje vlasništva fajlova, ali **samo u kontekstu svog korisničkog namespace-a**.

Na primer, kada imate sposobnost `CAP_SYS_ADMIN` unutar korisničkog namespace-a, možete izvršavati operacije koje obično zahtevaju ovu sposobnost, poput montiranja fajl sistema, ali samo u kontekstu svog korisničkog namespace-a. Sve operacije koje izvršite sa ovom sposobnošću neće uticati na host sistem ili druge namespace-ove.

{% hint style="warning" %}
Stoga, čak i ako dobijete novi proces unutar novog korisničkog namespace-a **dobijate sve sposobnosti nazad** (CapEff: 000001ffffffffff), zapravo možete **koristiti samo one koje su povezane sa namespace-om** (na primer, montiranje), ali ne sve. Dakle, samo to nije dovoljno da biste pobegli iz Docker kontejnera.
{% endhint %}

\`\`\`bash # There are the syscalls that are filtered after changing User namespace with: unshare -UmCpf bash

Probando: 0x067 . . . Error Probando: 0x070 . . . Error Probando: 0x074 . . . Error Probando: 0x09b . . . Error Probando: 0x0a3 . . . Error Probando: 0x0a4 . . . Error Probando: 0x0a7 . . . Error Probando: 0x0a8 . . . Error Probando: 0x0aa . . . Error Probando: 0x0ab . . . Error Probando: 0x0af . . . Error Probando: 0x0b0 . . . Error Probando: 0x0f6 . . . Error Probando: 0x12c . . . Error Probando: 0x130 . . . Error Probando: 0x139 . . . Error Probando: 0x140 . . . Error Probando: 0x141 . . . Error Probando: 0x143 . . . Error

```
## Reference
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
```
