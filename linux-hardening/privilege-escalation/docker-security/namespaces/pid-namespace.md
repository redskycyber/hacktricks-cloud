# PID Namespace

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

PID (Process IDentifier) namespace je funkcionalnost u Linux kernelu koja omogućava izolaciju procesa omogućavajući grupi procesa da ima svoj set jedinstvenih PID-ova, odvojenih od PID-ova u drugim namespace-ima. Ovo je posebno korisno u kontejnerizaciji, gde je izolacija procesa ključna za bezbednost i upravljanje resursima.

Kada se kreira novi PID namespace, prvom procesu u tom namespace-u se dodeljuje PID 1. Taj proces postaje "init" proces novog namespace-a i odgovoran je za upravljanje ostalim procesima unutar namespace-a. Svaki sledeći proces kreiran unutar namespace-a će imati jedinstveni PID unutar tog namespace-a, i ovi PID-ovi će biti nezavisni od PID-ova u drugim namespace-ima.

Iz perspektive procesa unutar PID namespace-a, on može videti samo druge procese u istom namespace-u. Nije svestan procesa u drugim namespace-ima i ne može da interaguje sa njima koristeći tradicionalne alate za upravljanje procesima (npr. `kill`, `wait`, itd.). Ovo pruža nivo izolacije koji pomaže u sprečavanju međusobnog ometanja procesa.

### Kako radi:

1. Kada se kreira novi proces (npr. korišćenjem `clone()` sistemskog poziva), proces može biti dodeljen novom ili postojećem PID namespace-u. **Ako se kreira novi namespace, proces postaje "init" proces tog namespace-a**.
2. **Kernel** održava **mapiranje između PID-ova u novom namespace-u i odgovarajućih PID-ova** u roditeljskom namespace-u (tj. namespace-u iz kojeg je novi namespace kreiran). Ovo mapiranje **omogućava kernelu da prevodi PID-ove kada je to potrebno**, kao što je slanje signala između procesa u različitim namespace-ima.
3. **Procesi unutar PID namespace-a mogu videti i interagovati samo sa drugim procesima u istom namespace-u**. Nisu svesni procesa u drugim namespace-ima, a njihovi PID-ovi su jedinstveni unutar njihovog namespace-a.
4. Kada se **PID namespace uništi** (npr. kada "init" proces namespace-a izađe), **svi procesi unutar tog namespace-a se terminiraju**. Ovo osigurava da se svi resursi povezani sa namespace-om pravilno očiste.

## Lab:

### Kreiranje različitih Namespace-ova

#### CLI

```bash
sudo unshare -pf --mount-proc /bin/bash
```

<details>

<summary>Greška: bash: fork: Ne može se alocirati memorija</summary>

Kada se `unshare` izvršava bez opcije `-f`, javlja se greška zbog načina na koji Linux obrađuje nove PID (Process ID) namespace-ove. Ključni detalji i rešenje su opisani u nastavku:

1. **Objašnjenje problema**:

* Linux kernel omogućava procesu da kreira nove namespace-ove koristeći `unshare` sistemski poziv. Međutim, proces koji pokreće kreiranje novog PID namespace-a (nazvan "unshare" proces) ne ulazi u novi namespace; samo njegovi podprocesi to čine.
* Pokretanje `%unshare -p /bin/bash%` pokreće `/bin/bash` u istom procesu kao i `unshare`. Kao rezultat, `/bin/bash` i njegovi podprocesi se nalaze u originalnom PID namespace-u.
* Prvi podproces `/bin/bash` u novom namespace-u postaje PID 1. Kada ovaj proces završi, pokreće se čišćenje namespace-a ako nema drugih procesa, jer PID 1 ima posebnu ulogu usvajanja siročadi. Linux kernel tada onemogućava alokaciju PID-ova u tom namespace-u.

2. **Posledica**:

* Izlazak PID 1 iz novog namespace-a dovodi do čišćenja `PIDNS_HASH_ADDING` zastavice. To rezultira neuspehom funkcije `alloc_pid` pri alociranju novog PID-a prilikom kreiranja novog procesa, što dovodi do greške "Ne može se alocirati memorija".

3. **Rešenje**:

* Problem se može rešiti korišćenjem opcije `-f` sa `unshare`. Ova opcija čini da `unshare` fork-uje novi proces nakon kreiranja novog PID namespace-a.
* Izvršavanje `%unshare -fp /bin/bash%` osigurava da sam `unshare` komanda postane PID 1 u novom namespace-u. `/bin/bash` i njegovi podprocesi su tada sigurno smešteni unutar ovog novog namespace-a, sprečavajući prevremeni izlazak PID 1 i omogućavajući normalnu alokaciju PID-ova.

Osiguravanjem da `unshare` radi sa opcijom `-f`, novi PID namespace se pravilno održava, omogućavajući `/bin/bash` i njegovim podprocesima da rade bez greške alokacije memorije.

</details>

Montiranjem nove instance `/proc` fajl sistema, ako koristite parametar `--mount-proc`, obezbeđujete da novi mount namespace ima **tačan i izolovan prikaz informacija o procesima specifičnim za taj namespace**.

#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

### Proverite u kojem namespace-u se nalazi vaš proces

Da biste proverili u kojem namespace-u se nalazi vaš proces, možete koristiti sledeću komandu:

```bash
ls -l /proc/$$/ns
```

Ova komanda će vam prikazati sve namespace-ove u kojima se trenutno nalazi vaš proces.

```bash
ls -l /proc/self/ns/pid
lrwxrwxrwx 1 root root 0 Apr  3 18:45 /proc/self/ns/pid -> 'pid:[4026532412]'
```

### Pronađite sve PID namespace-ove

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name pid -exec readlink {} \; 2>/dev/null | sort -u
```
{% endcode %}

Imajte na umu da korisnik sa root privilegijama iz početnog (podrazumevanog) PID namespace-a može videti sve procese, čak i one u novim PID namespace-ima, zbog čega možemo videti sve PID namespace-e.

### Uđite unutar PID namespace-a

```bash
nsenter -t TARGET_PID --pid /bin/bash
```

Kada uđete unutar PID namespace-a iz zadanih namespace-a, i dalje ćete moći vidjeti sve procese. I proces iz tog PID ns-a će moći vidjeti novi bash na PID ns-u.

Takođe, možete **ući u drugi PID namespace samo ako ste root**. I **ne možete** **ući** u drugi namespace **bez deskriptora** koji na njega pokazuje (poput `/proc/self/ns/pid`)

## Reference

* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite vidjeti **oglašavanje vaše kompanije u HackTricks-u** ili **preuzeti HackTricks u PDF formatu** Provjerite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podijelite svoje hakirajuće trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
