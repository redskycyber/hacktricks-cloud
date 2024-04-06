# Abusing Docker Socket for Privilege Escalation

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Postoje situacije kada imate **pristup Docker Socket-u** i želite ga iskoristiti za **eskalciju privilegija**. Neke radnje mogu biti veoma sumnjive i možda želite da ih izbegnete, pa ovde možete pronaći različite zastavice koje mogu biti korisne za eskalaciju privilegija:

### Preko montiranja

Možete **montirati** različite delove **fajl sistema** u kontejneru koji se izvršava kao root i **pristupiti** im.

* **`-v /:/host`** -> Montirajte fajl sistem domaćina u kontejneru kako biste mogli **čitati fajl sistem domaćina**.
* Ako želite da se **osećate kao da ste na domaćinu**, ali da budete u kontejneru, možete onemogućiti druge mehanizme odbrane koristeći zastavice kao što su:
* `--privileged`
* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `-security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* \*\*`--device=/dev/sda1 --cap-add=SYS_ADMIN --security-opt apparmor=unconfined` \*\* -> Ovo je slično prethodnoj metodi, ali ovde **montiramo disk uređaj**. Zatim, unutar kontejnera pokrenite `mount /dev/sda1 /mnt` i možete **pristupiti** fajl sistemu domaćina u `/mnt`
* Pokrenite `fdisk -l` na domaćinu da biste pronašli uređaj `</dev/sda1>` koji treba montirati
* **`-v /tmp:/host`** -> Ako iz nekog razloga možete **samo montirati neki direktorijum** sa domaćina i imate pristup unutar domaćina. Montirajte ga i kreirajte **`/bin/bash`** sa **suid** u montiranom direktorijumu kako biste ga mogli **izvršiti sa domaćina i eskalirati privilegije do root-a**.

{% hint style="info" %}
Imajte na umu da možda ne možete montirati direktorijum `/tmp`, ali možete montirati **drugi direktorijum za pisanje**. Možete pronaći direktorijume za pisanje koristeći: `find / -writable -type d 2>/dev/null`

**Imajte na umu da ne svi direktorijumi na Linux mašini podržavaju suid bit!** Da biste proverili koji direktorijumi podržavaju suid bit, pokrenite `mount | grep -v "nosuid"`. Na primer, obično `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` i `/var/lib/lxcfs` ne podržavaju suid bit.

Takođe imajte na umu da ako možete **montirati `/etc`** ili bilo koji drugi direktorijum **koji sadrži konfiguracione fajlove**, možete ih promeniti iz Docker kontejnera kao root kako biste ih **zloupotrebili na domaćinu** i eskalirali privilegije (možda izmenom `/etc/shadow`)
{% endhint %}

### Bekstvo iz kontejnera

* **`--privileged`** -> Sa ovom zastavicom [uklanjate izolaciju iz kontejnera](docker-privileged.md#what-affects). Pogledajte tehnike za [bekstvo iz privilegovanih kontejnera kao root](docker-breakout-privilege-escalation/#automatic-enumeration-and-escape).
* **`--cap-add=<CAPABILITY/ALL> [--security-opt apparmor=unconfined] [--security-opt seccomp=unconfined] [-security-opt label:disable]`** -> Da biste [eskaliarali privilegije zloupotrebom mogućnosti](../linux-capabilities.md), **dodelite tu mogućnost kontejneru** i onemogućite druge metode zaštite koje mogu sprečiti iskorišćavanje.

Curl

Na ovoj stranici smo razgovarali o načinima eskalacije privilegija koristeći Docker zastavice, možete pronaći **načine za zloupotrebu ovih metoda koristeći curl** komandu na stranici:

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
