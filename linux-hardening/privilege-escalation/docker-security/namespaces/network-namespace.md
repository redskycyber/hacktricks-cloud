# Network Namespace

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

Mrežni namespace je funkcionalnost Linux kernela koja omogućava izolaciju mrežnog sloja, omogućavajući **svakom mrežnom namespace-u da ima sopstvenu nezavisnu mrežnu konfiguraciju**, interfejse, IP adrese, rutne tabele i pravila za zaštitu od požara. Ova izolacija je korisna u raznim scenarijima, kao što je kontejnerizacija, gde svaki kontejner treba da ima sopstvenu mrežnu konfiguraciju, nezavisnu od drugih kontejnera i host sistema.

### Kako radi:

1. Kada se kreira novi mrežni namespace, on počinje sa **potpuno izolovanim mrežnim slojem**, sa **bez mrežnih interfejsa** osim petljačkog interfejsa (lo). To znači da procesi koji se izvršavaju u novom mrežnom namespace-u ne mogu komunicirati sa procesima u drugim namespace-ima ili host sistemu po default-u.
2. **Virtuelni mrežni interfejsi**, kao što su veth parovi, mogu se kreirati i premestiti između mrežnih namespace-ova. Ovo omogućava uspostavljanje mrežne konekcije između namespace-ova ili između namespace-a i host sistema. Na primer, jedan kraj veth para može biti smešten u mrežnom namespace-u kontejnera, a drugi kraj može biti povezan sa **mostom** ili drugim mrežnim interfejsom u host namespace-u, pružajući mrežnu konekciju kontejneru.
3. Mrežni interfejsi unutar namespace-a mogu imati **svoje IP adrese, rutne tabele i pravila za zaštitu od požara**, nezavisno od drugih namespace-a. Ovo omogućava procesima u različitim mrežnim namespace-ima da imaju različite mrežne konfiguracije i da funkcionišu kao da se izvršavaju na odvojenim mrežnim sistemima.
4. Procesi mogu da se premeštaju između namespace-a koristeći `setns()` sistemski poziv, ili da kreiraju nove namespace-e koristeći `unshare()` ili `clone()` sistemski poziv sa `CLONE_NEWNET` zastavicom. Kada proces pređe u novi namespace ili ga kreira, počeće da koristi mrežnu konfiguraciju i interfejse povezane sa tim namespace-om.

## Lab:

### Kreiranje različitih Namespace-ova

#### CLI

```bash
sudo unshare -n [--mount-proc] /bin/bash
# Run ifconfig or ip -a
```

Montiranjem nove instance `/proc` fajl sistema, korišćenjem parametra `--mount-proc`, obezbeđujete da nova namespace za montiranje ima **tačan i izolovan prikaz informacija o procesu specifičnih za tu namespace**.

<details>

<summary>Greška: bash: fork: Ne može se alocirati memorija</summary>

Kada se `unshare` izvršava bez opcije `-f`, javlja se greška zbog načina na koji Linux rukuje novim PID (Process ID) namespace-om. Ključni detalji i rešenje su opisani u nastavku:

1. **Objašnjenje problema**:

* Linux kernel omogućava procesu da kreira nove namespace-ove koristeći `unshare` sistemski poziv. Međutim, proces koji pokreće kreiranje novog PID namespace-a (nazvan "unshare" proces) ne ulazi u novi namespace; samo njegovi podprocesi to čine.
* Pokretanje `%unshare -p /bin/bash%` pokreće `/bin/bash` u istom procesu kao i `unshare`. Kao rezultat, `/bin/bash` i njegovi podprocesi su u originalnom PID namespace-u.
* Prvi podproces `/bin/bash` u novom namespace-u postaje PID 1. Kada ovaj proces završi, pokreće se čišćenje namespace-a ako nema drugih procesa, jer PID 1 ima posebnu ulogu usvajanja siročadi. Linux kernel će tada onemogućiti alokaciju PID-a u tom namespace-u.

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
# Run ifconfig or ip -a
```

### Proverite u kojem namespace-u se nalazi vaš proces

Da biste proverili u kojem namespace-u se nalazi vaš proces, možete koristiti sledeću komandu:

```bash
ls -l /proc/$$/ns/net
```

Ova komanda će vam prikazati simboličku vezu koja pokazuje na trenutni namespace mreže u kojem se nalazi vaš proces.

```bash
ls -l /proc/self/ns/net
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/net -> 'net:[4026531840]'
```

### Pronađi sve mrežne namespace-ove

{% code overflow="wrap" %}
```
```
{% endcode %}

```bash
sudo find /proc -maxdepth 3 -type l -name net -exec readlink {} \; 2>/dev/null | sort -u | grep "net:"
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name net -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```

```
```

\`\`\`bash nsenter -n TARGET\_PID --pid /bin/bash \`\`\` Takođe, možete \*\*ući u drugi proces namespace samo ako ste root\*\*. I \*\*ne možete\*\* \*\*ući\*\* u drugi namespace \*\*bez deskriptora\*\* koji na njega pokazuje (poput \`/proc/self/ns/net\`).

## Reference

* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite videti **oglašavanje vaše kompanije u HackTricks-u** ili **preuzeti HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
