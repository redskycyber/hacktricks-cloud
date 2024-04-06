# Docker Security

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako izgradite i **automatizujete radne tokove** pokretane najnaprednijim alatima zajednice.\
Dobijte pristup danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## **Osnovna bezbednost Docker Engine-a**

**Docker engine** koristi Linux kernel-ove **Namespaces** i **Cgroups** za izolaciju kontejnera, pružajući osnovni nivo bezbednosti. Dodatna zaštita se obezbeđuje kroz **Capabilities dropping**, **Seccomp**, i **SELinux/AppArmor**, poboljšavajući izolaciju kontejnera. **Auth plugin** može dodatno ograničiti korisničke akcije.

![Docker Security](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### Bezbedan pristup Docker Engine-u

Docker engine se može pristupiti lokalno putem Unix soketa ili udaljeno korišćenjem HTTP-a. Za udaljeni pristup, bitno je koristiti HTTPS i **TLS** kako bi se osigurala poverljivost, integritet i autentifikacija.

Docker engine, po podrazumevanim podešavanjima, osluškuje Unix soket na `unix:///var/run/docker.sock`. Na Ubuntu sistemima, Docker-ova startna podešavanja se definišu u `/etc/default/docker`. Da biste omogućili udaljeni pristup Docker API-ju i klijentu, izložite Docker daemon preko HTTP soketa dodavanjem sledećih podešavanja:

```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```

Međutim, izlaganje Docker demona preko HTTP-a nije preporučljivo zbog sigurnosnih razloga. Preporučljivo je obezbediti veze korišćenjem HTTPS-a. Postoje dva glavna pristupa obezbeđivanju veze:

1. Klijent proverava identitet servera.
2. Klijent i server međusobno autentično proveravaju identitet.

Sertifikati se koriste za potvrdu identiteta servera. Za detaljne primere oba metoda, pogledajte [**ovaj vodič**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/).

### Bezbednost slika kontejnera

Slike kontejnera mogu se čuvati u privatnim ili javnim repozitorijumima. Docker nudi nekoliko opcija za čuvanje slika kontejnera:

* [**Docker Hub**](https://hub.docker.com): Javna usluga registra od strane Dockera.
* [**Docker Registry**](https://github.com/docker/distribution): Projekat otvorenog koda koji korisnicima omogućava da hostuju svoj sopstveni registar.
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): Komercijalna ponuda Dockera, sa autentifikacijom korisnika zasnovanom na ulogama i integracijom sa LDAP direktorijumskim servisima.

### Skeniranje slika

Kontejneri mogu imati **sigurnosne ranjivosti** ili zbog osnovne slike ili zbog softvera instaliranog na vrhu osnovne slike. Docker radi na projektu pod nazivom **Nautilus** koji vrši sigurnosno skeniranje kontejnera i navodi ranjivosti. Nautilus radi tako što upoređuje svaki sloj slike kontejnera sa repozitorijumom ranjivosti kako bi identifikovao sigurnosne propuste.

Za više [**informacija pročitajte ovo**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

Komanda **`docker scan`** omogućava skeniranje postojećih Docker slika korišćenjem imena ili ID slike. Na primer, pokrenite sledeću komandu da skenirate sliku hello-world:

```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```

* [**`trivy`**](https://github.com/aquasecurity/trivy)

```bash
trivy -q -f json <container_name>:<tag>
```

* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)

```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```

* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)

```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```

### Potpisivanje Docker slike

Potpisivanje Docker slike osigurava sigurnost i integritet slika korišćenih u kontejnerima. Evo sažetog objašnjenja:

* **Docker Content Trust** koristi Notary projekat, zasnovan na The Update Framework (TUF), za upravljanje potpisivanjem slika. Za više informacija, pogledajte [Notary](https://github.com/docker/notary) i [TUF](https://theupdateframework.github.io).
* Da biste aktivirali Docker content trust, postavite `export DOCKER_CONTENT_TRUST=1`. Ova funkcija je isključena po podrazumevanju u Docker verziji 1.10 i kasnijim verzijama.
* Sa ovom funkcijom omogućenom, samo potpisane slike mogu biti preuzete. Početno slanje slike zahteva postavljanje lozinki za korenske i označene ključeve, pri čemu Docker takođe podržava Yubikey za unapređenu sigurnost. Više detalja možete pronaći [ovde](https://blog.docker.com/2015/11/docker-content-trust-yubikey/).
* Pokušaj preuzimanja nepotpisane slike sa omogućenim content trust-om rezultira greškom "No trust data for latest".
* Za slanje slika nakon prvog, Docker traži lozinku ključa repozitorijuma za potpisivanje slike.

Za bekapovanje vaših privatnih ključeva, koristite komandu:

```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```

Kada prebacujete Docker hostove, potrebno je premestiti root i repozitorijum ključeve kako biste održali operacije.

***

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da biste lako izgradili i **automatizovali radne tokove** pokretane najnaprednijim alatima zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Funkcije Sigurnosti Kontejnera

<details>

<summary>Rezime funkcija sigurnosti kontejnera</summary>

**Glavne funkcije izolacije glavnog procesa**

U kontejnerizovanim okruženjima, izolacija projekata i njihovih procesa je od suštinskog značaja za sigurnost i upravljanje resursima. Evo pojednostavljenog objašnjenja ključnih koncepata:

**Prostori imena (Namespaces)**

* **Svrha**: Osigurati izolaciju resursa poput procesa, mreže i fajl sistema. Posebno u Docker-u, prostori imena čuvaju procese kontejnera odvojene od domaćina i drugih kontejnera.
* **Korišćenje `unshare`**: Komanda `unshare` (ili odgovarajući sistemski poziv) se koristi za kreiranje novih prostora imena, pružajući dodatni sloj izolacije. Međutim, iako Kubernetes inherentno ne blokira ovo, Docker to čini.
* **Ograničenje**: Kreiranje novih prostora imena ne dozvoljava procesu da se vrati na podrazumevane prostore imena domaćina. Da bi prodro u prostore imena domaćina, obično bi bio potreban pristup direktorijumu `/proc` domaćina, koristeći `nsenter` za ulaz.

**Grupa Kontrola (CGroups)**

* **Funkcija**: Prvenstveno se koristi za dodelu resursa među procesima.
* **Aspekt sigurnosti**: Same CGroups ne nude sigurnost izolacije, osim funkcije `release_agent`, koja, ako nije ispravno konfigurisana, potencijalno može biti iskorišćena za neovlašćeni pristup.

**Odbacivanje sposobnosti (Capability Drop)**

* **Važnost**: To je ključna sigurnosna funkcija za izolaciju procesa.
* **Funkcionalnost**: Ona ograničava akcije koje root proces može izvršiti odbacivanjem određenih sposobnosti. Čak i ako proces radi sa privilegijama root-a, nedostatak neophodnih sposobnosti sprečava ga da izvršava privilegovane akcije, jer će sistemski pozivi propasti zbog nedovoljnih dozvola.

Ovo su **preostale sposobnosti** nakon što proces odbaci ostale:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

Podrazumevano je omogućeno u Dockeru. Pomaže da se **još više ograniče syscalls** koje proces može pozvati.\
**Podrazumevani Docker Seccomp profil** može se pronaći na [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**AppArmor**

Docker ima predložak koji možete aktivirati: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

Ovo će omogućiti smanjenje mogućnosti, syscalls, pristup fajlovima i fasciklama...

</details>

### Namespaces

**Namespaces** su funkcija Linux kernela koja **deli resurse kernela** tako da jedan skup **procesa vidi** jedan skup **resursa**, dok **drugi** skup **procesa** vidi **drugi** skup resursa. Ova funkcija radi tako što ima isti namespace za skup resursa i procesa, ali ti namespace-ovi se odnose na različite resurse. Resursi mogu postojati u više prostora.

Docker koristi sledeće Linux kernel Namespaces za postizanje izolacije kontejnera:

* pid namespace
* mount namespace
* network namespace
* ipc namespace
* UTS namespace

Za **više informacija o namespace-ovima** pogledajte sledeću stranicu:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

Linux kernel funkcija **cgroups** omogućava mogućnost **ograničavanja resursa kao što su cpu, memorija, io, propusnost mreže među** skupom procesa. Docker omogućava kreiranje kontejnera koristeći cgroup funkciju koja omogućava kontrolu resursa za određeni kontejner.\
Sledeći je kontejner kreiran sa ograničenjem memorije korisničkog prostora na 500m, ograničenjem kernel memorije na 50m, deljenjem cpu na 512, blkioweight na 400. CPU deljenje je odnos koji kontroliše upotrebu CPU-a kontejnera. Podrazumevana vrednost je 1024, a opseg je između 0 i 1024. Ako tri kontejnera imaju isto CPU deljenje od 1024, svaki kontejner može koristiti do 33% CPU-a u slučaju sukoba resursa CPU-a. blkio-weight je odnos koji kontroliše IO kontejnera. Podrazumevana vrednost je 500, a opseg je između 10 i 1000.

```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```

Da biste dobili cgroup kontejnera, možete uraditi:

```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```

Za više informacija pogledajte:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Mogućnosti

Mogućnosti omogućavaju **finiju kontrolu mogućnosti koje se mogu dozvoliti** za korisnika root. Docker koristi mogućnost funkcije jezgra Linux-a da **ograniči operacije koje se mogu obaviti unutar kontejnera** bez obzira na vrstu korisnika.

Kada se pokrene docker kontejner, **proces odbacuje osetljive mogućnosti koje bi proces mogao koristiti za bekstvo iz izolacije**. Ovo pokušava da osigura da proces neće moći da obavlja osetljive radnje i bekstva:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Seccomp u Docker-u

Ovo je sigurnosna funkcija koja omogućava Docker-u da **ograniči syscalls** koji se mogu koristiti unutar kontejnera:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### AppArmor u Docker-u

**AppArmor** je poboljšanje jezgra za ograničavanje **kontejnera** na **ograničen** skup **resursa** sa **profilima po programu**:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### SELinux u Docker-u

* **Sistem označavanja**: SELinux dodeljuje jedinstvenu oznaku svakom procesu i objektu datotečnog sistema.
* **Sprovođenje politike**: Sprovodi sigurnosne politike koje definišu koje radnje oznaka procesa može obavljati na drugim oznakama unutar sistema.
* **Oznake procesa kontejnera**: Kada motori kontejnera pokrenu procese kontejnera, obično im se dodeljuje ograničena SELinux oznaka, obično `container_t`.
* **Označavanje datoteka unutar kontejnera**: Datoteke unutar kontejnera obično su označene kao `container_file_t`.
* **Pravila politike**: SELinux politika pretežno osigurava da procesi sa oznakom `container_t` mogu samo da interaguju (čitaju, pišu, izvršavaju) sa datotekama označenim kao `container_file_t`.

Ovaj mehanizam osigurava da čak i ako je proces unutar kontejnera kompromitovan, ograničen je na interakciju samo sa objektima koji imaju odgovarajuće oznake, značajno ograničavajući potencijalnu štetu od takvih kompromitovanja.

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

U Docker-u, autorizacioni dodatak igra ključnu ulogu u sigurnosti odlučujući da li da dozvoli ili blokira zahteve ka Docker demonu. Ova odluka se donosi ispitivanjem dva ključna konteksta:

* **Kontekst autentifikacije**: Ovde se nalaze sveobuhvatne informacije o korisniku, kao što su ko su i kako su se autentifikovali.
* **Kontekst komande**: Ovaj sadrži sve relevantne podatke vezane za zahtev koji se pravi.

Ovi konteksti pomažu da se osigura da se obrađuju samo legitimni zahtevi od autentifikovanih korisnika, unapređujući sigurnost Docker operacija.

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## DoS iz kontejnera

Ako pravilno ne ograničavate resurse koje kontejner može koristiti, kompromitovan kontejner može DoS-ovati host na kojem se izvršava.

* CPU DoS

```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```

* Napad na propusnost (Bandwidth DoS)

```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```

## Zanimljive Docker zastavice

### --privileged zastavica

Na sledećoj stranici možete saznati **šta podrazumeva zastavica `--privileged`**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

Ako pokrećete kontejner gde napadač uspe da pristupi kao korisnik sa niskim privilegijama. Ako imate **pogrešno konfigurisan suid binarni fajl**, napadač može zloupotrebiti to i **povećati privilegije unutar** kontejnera. Što mu može omogućiti da pobegne iz njega.

Pokretanje kontejnera sa opcijom **`no-new-privileges`** omogućiće **sprečavanje ovakvog povećanja privilegija**.

```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```

#### Ostalo

```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```

Za više **`--security-opt`** opcija proverite: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## Ostale bezbednosne razmatranja

### Upravljanje tajnama: Najbolje prakse

Važno je izbegavati ugradnju tajni direktno u Docker slike ili korišćenje okružnih promenljivih, jer ovi metodi izlažu vaše osetljive informacije svakome ko ima pristup kontejneru putem komandi poput `docker inspect` ili `exec`.

**Docker volumeni** su sigurnija alternativa, preporučena za pristup osetljivim informacijama. Mogu se koristiti kao privremeni fajl sistem u memoriji, smanjujući rizike povezane sa `docker inspect` i logovanjem. Međutim, korisnici sa administratorskim pravima i oni sa `exec` pristupom kontejneru i dalje mogu pristupiti tajnama.

**Docker tajne** nude još sigurniji metod za rukovanje osetljivim informacijama. Za slučajeve koji zahtevaju tajne tokom faze izgradnje slike, **BuildKit** predstavlja efikasno rešenje sa podrškom za tajne tokom izgradnje, poboljšavajući brzinu izgradnje i pružajući dodatne funkcije.

Da biste iskoristili BuildKit, može se aktivirati na tri načina:

1. Putem okružne promenljive: `export DOCKER_BUILDKIT=1`
2. Dodavanjem prefiksa komandama: `DOCKER_BUILDKIT=1 docker build .`
3. Omogućavanjem kao podrazumevano u Docker konfiguraciji: `{ "features": { "buildkit": true } }`, praćeno restartovanjem Docker-a.

BuildKit omogućava korišćenje tajni tokom izgradnje sa opcijom `--secret`, obezbeđujući da ove tajne nisu uključene u keš izgradnje slike ili konačnu sliku, korišćenjem komande poput:

```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```

Za tajne potrebne u pokrenutom kontejneru, **Docker Compose i Kubernetes** nude robustna rešenja. Docker Compose koristi ključ `secrets` u definiciji servisa za specificiranje tajnih fajlova, kako je prikazano u primeru `docker-compose.yml`:

```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```

Ova konfiguracija omogućava korišćenje tajni prilikom pokretanja usluga pomoću Docker Compose-a.

U Kubernetes okruženjima, tajne su podržane na nivou platforme i mogu se dalje upravljati alatima poput [Helm-Secrets](https://github.com/futuresimple/helm-secrets). Upravljanje tajnama u Kubernetes-u putem Role Based Access Controls (RBAC) poboljšava sigurnost upravljanja tajnama, slično kao u Docker Enterprise-u.

### gVisor

**gVisor** je aplikaciono jezgro napisano u Go-u, koje implementira značajan deo Linux sistemskog interfejsa. Uključuje [Open Container Initiative (OCI)](https://www.opencontainers.org) runtime nazvan `runsc` koji pruža **izolacionu granicu između aplikacije i host kernela**. Runtime `runsc` se integriše sa Dockerom i Kubernetes-om, čineći jednostavnim pokretanje kontejnera u pesku.

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** je zajednica otvorenog koda koja radi na izgradnji sigurnog runtime-a kontejnera sa lakim virtuelnim mašinama koje se ponašaju i izvode kao kontejneri, ali pružaju **jaču izolaciju radnog opterećenja korišćenjem tehnologije hardverske virtualizacije** kao drugog sloja odbrane.

{% embed url="https://katacontainers.io/" %}

### Saveti ukratko

* **Ne koristite `--privileged` zastavicu ili montirajte** [**Docker socket unutar kontejnera**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**.** Docker socket omogućava pokretanje kontejnera, pa je to jednostavan način da se preuzme potpuna kontrola nad hostom, na primer, pokretanjem drugog kontejnera sa `--privileged` zastavicom.
* Ne pokrećite kao root unutar kontejnera. Koristite **različitog korisnika** i **user namespaces**. Root u kontejneru je isti kao na hostu osim ako nije preusmeren pomoću user namespaces-a. On je samo delimično ograničen, pretežno, Linux namespaces-ima, mogućnostima i cgroups-ima.
* [**Odbacite sve mogućnosti**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) i omogućite samo one koje su potrebne** (`--cap-add=...`). Mnogi radni opterećenja ne zahtevaju nikakve mogućnosti, a dodavanje istih povećava obim potencijalnog napada.
* [**Koristite sigurnosnu opciju “no-new-privileges”**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) da sprečite procese da steknu više privilegija, na primer putem suid binarnih fajlova.
* [**Ograničite resurse dostupne kontejneru**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**.** Ograničenja resursa mogu zaštititi mašinu od napada uskraćivanjem usluge.
* **Prilagodite** profile [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**,** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(ili SELinux)** da ograniče radnje i sistemske pozive dostupne kontejneru na minimum potreban.
* **Koristite** [**zvanične Docker slike**](https://docs.docker.com/docker-hub/official\_images/) **i zahtevajte potpise** ili izgradite svoje zasnovane na njima. Ne nasleđujte ili ne koristite [slike sa zadnjim vratima](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/). Takođe čuvajte root ključeve, lozinke na sigurnom mestu. Docker ima planove za upravljanje ključevima sa UCP-om.
* **Redovno** **ponovo izgradite** svoje slike da **primenite sigurnosne zakrpe na hostu i slikama.**
* Mudro upravljajte **svojim tajnama** tako da je teško napadaču da im pristupi.
* Ako **izlažete docker daemon koristite HTTPS** sa autentifikacijom klijenta i servera.
* U svom Dockerfile-u, **favorizujte COPY umesto ADD**. ADD automatski raspakuje zipovane fajlove i može kopirati fajlove sa URL-ova. COPY nema ove mogućnosti. Kad god je moguće, izbegavajte korišćenje ADD kako ne biste bili podložni napadima putem udaljenih URL-ova i Zip fajlova.
* Imajte **odvojene kontejnere za svaku mikro-s**ervisu
* **Ne stavljajte ssh** unutar kontejnera, “docker exec” se može koristiti za ssh ka kontejneru.
* Imajte **manje** slike **kontejnera**

## Bekstvo iz Docker-a / Eskalacija privilegija

Ako ste **unutar Docker kontejnera** ili imate pristup korisniku u **docker grupi**, možete pokušati **pobegnuti i eskalirati privilegije**:

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Bypassovanje Docker Authentication Plugin-a

Ako imate pristup docker socket-u ili pristup korisniku u **docker grupi ali vaše akcije su ograničene Docker auth plugin-om**, proverite da li ga možete **bypassovati:**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Jačanje Docker-a

* Alat [**docker-bench-security**](https://github.com/docker/docker-bench-security) je skripta koja proverava desetine uobičajenih najboljih praksi oko implementacije Docker kontejnera u produkciji. Testovi su svi automatizovani i zasnovani na [CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/).\
  Morate pokrenuti alat sa hosta koji pokreće Docker ili iz kontejnera sa dovoljno privilegija. Saznajte **kako ga pokrenuti u README-u:** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security).

## Reference

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/\_fel1x/status/1151487051986087936](https://twitter.com/\_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/)
* [https://en.wikipedia.org/wiki/Linux\_namespaces](https://en.wikipedia.org/wiki/Linux\_namespaces)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)
* [https://docs.docker.com/engine/extend/plugins\_authorization](https://docs.docker.com/engine/extend/plugins\_authorization)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/](https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/)

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako izgradite i **automatizujete radne tokove** pokretane najnaprednijim alatima zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
