# CGroups

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

**Linux Control Groups**, ili **cgroups**, su funkcionalnost Linux kernela koja omogućava alokaciju, ograničavanje i prioritetizaciju sistemskih resursa kao što su CPU, memorija i disk I/O među grupama procesa. Oni pružaju mehanizam za **upravljanje i izolaciju korišćenja resursa** kolekcija procesa, korisnih za svrhe kao što su ograničavanje resursa, izolacija radnog opterećenja i prioritetizacija resursa među različitim grupama procesa.

Postoje **dve verzije cgroups-a**: verzija 1 i verzija 2. Obe mogu biti istovremeno korišćene na sistemu. Osnovna razlika je da **cgroups verzija 2** uvodi **hijerarhijsku strukturu nalik stablu**, omogućavajući detaljniju raspodelu resursa među grupama procesa. Pored toga, verzija 2 donosi razne poboljšanja, uključujući:

Pored nove hijerarhijske organizacije, cgroups verzija 2 takođe je uvela **nekoliko drugih promena i poboljšanja**, kao što je podrška za **nove kontrolere resursa**, bolja podrška za legacy aplikacije i poboljšana performansa.

Ukupno gledano, cgroups **verzija 2 nudi više funkcionalnosti i bolju performansu** od verzije 1, ali ova poslednja se i dalje može koristiti u određenim scenarijima gde je kompatibilnost sa starijim sistemima bitna.

Možete izlistati v1 i v2 cgroups za bilo koji proces tako što ćete pogledati njegov cgroup fajl u /proc/\<pid>. Možete početi tako što ćete pogledati cgroups vaše shell-a sa ovom komandom:

```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```

Struktura izlaza je sledeća:

* **Brojevi 2-12**: cgroups v1, pri čemu svaka linija predstavlja drugačiji cgroup. Kontroleri za ove su navedeni pored broja.
* **Broj 1**: Takođe cgroups v1, ali samo u svrhu upravljanja (postavljen od strane, na primer, systemd-a) i nema kontrolera.
* **Broj 0**: Predstavlja cgroups v2. Nema navedenih kontrolera i ova linija je ekskluzivna za sisteme koji koriste samo cgroups v2.
* **Imena su hijerarhijska**, slična putanjama datoteka, što ukazuje na strukturu i odnos između različitih cgroup-ova.
* **Imena poput /user.slice ili /system.slice** specificiraju kategorizaciju cgroup-ova, pri čemu je user.slice obično za prijavljene sesije koje upravlja systemd, a system.slice za sistemski servis.

### Pregledanje cgroup-ova

Datotečni sistem se obično koristi za pristupanje **cgroup-ovima**, odstupajući od tradicionalnog Unix sistemskog poziva koji se tradicionalno koristi za interakciju sa kernelom. Da biste istražili konfiguraciju cgroup-a ljuske, trebali biste pregledati datoteku **/proc/self/cgroup**, koja otkriva cgroup ljuske. Zatim, navigirajući do direktorijuma **/sys/fs/cgroup** (ili **`/sys/fs/cgroup/unified`**), i pronalaženjem direktorijuma koji deli ime cgroup-a, možete posmatrati različite postavke i informacije o korišćenju resursa relevantne za cgroup.

![Cgroup Filesystem](<../../../.gitbook/assets/image (10) (2) (2).png>)

Ključne datoteke interfejsa za cgroup-ove imaju prefiks **cgroup**. Datoteka **cgroup.procs**, koja se može pregledati standardnim komandama poput cat, navodi procese unutar cgroup-a. Druga datoteka, **cgroup.threads**, uključuje informacije o nitima.

![Cgroup Procs](<../../../.gitbook/assets/image (1) (1) (5).png>)

Cgroup-ovi koji upravljaju ljuskama obično obuhvataju dva kontrolera koji regulišu upotrebu memorije i broj procesa. Da biste interagovali sa kontrolerom, trebali biste se konsultovati sa datotekama koje nose prefiks kontrolera. Na primer, **pids.current** bi se koristio da bi se utvrdio broj niti u cgroup-u.

![Cgroup Memory](<../../../.gitbook/assets/image (3) (5).png>)

Indikacija **max** u vrednosti ukazuje na odsustvo specifičnog ograničenja za cgroup. Međutim, zbog hijerarhijske prirode cgroup-ova, ograničenja mogu biti nametnuta od strane cgroup-a na nižem nivou u hijerarhiji direktorijuma.

### Manipulacija i kreiranje cgroup-ova

Procesi se dodeljuju cgroup-ovima tako što se **upisuje njihov ID procesa (PID) u datoteku `cgroup.procs`**. Za ovo su potrebne privilegije root-a. Na primer, da biste dodali proces:

```bash
echo [pid] > cgroup.procs
```

Slično tome, **izmena atributa cgroup-a, poput postavljanja ograničenja PID-a**, se vrši pisanjem željene vrednosti u odgovarajući fajl. Da biste postavili maksimalno 3.000 PID-ova za cgroup:

```bash
echo 3000 > pids.max
```

**Kreiranje novih cgroups** podrazumeva pravljenje nove poddirektorijuma unutar hijerarhije cgroups, što podstiče kernel da automatski generiše neophodne interfejsne fajlove. Iako cgroups bez aktivnih procesa mogu biti uklonjeni pomoću `rmdir` komande, treba imati na umu određena ograničenja:

* **Procesi mogu biti smešteni samo u list cgroups** (tj. najugnježdenije u hijerarhiji).
* **Cgroup ne može imati kontroler koji ne postoji u roditeljskom cgroup-u**.
* **Kontroleri za pod-cgroups moraju biti eksplicitno deklarisani** u fajlu `cgroup.subtree_control`. Na primer, da biste omogućili CPU i PID kontrolere u pod-cgroup-u:

```bash
echo "+cpu +pids" > cgroup.subtree_control
```

**Root cgroup** je izuzetak od ovih pravila, omogućavajući direktno postavljanje procesa. To se može koristiti za uklanjanje procesa iz systemd upravljanja.

**Pracenje korišćenja CPU-a** unutar cgroup-a je moguće putem datoteke `cpu.stat`, koja prikazuje ukupno vreme CPU-a koje je potrošeno, korisno za praćenje korišćenja preko podprocesa servisa:

<figure><img src="../../../.gitbook/assets/image (2) (6) (3).png" alt=""><figcaption><p>Statistika korišćenja CPU-a prikazana u datoteci cpu.stat</p></figcaption></figure>

## Reference

* **Knjiga: Kako Linux radi, 3. izdanje: Šta svaki superkorisnik treba da zna, autora Brian Ward**

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
