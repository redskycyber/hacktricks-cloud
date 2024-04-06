# Checklist - Linux Privilege Escalation

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Pridružite se [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) serveru kako biste komunicirali sa iskusnim hakerima i lovcima na bagove!

**Hakerski uvidi**\
Uključite se u sadržaj koji istražuje uzbuđenje i izazove hakovanja

**Vesti o hakovanju u realnom vremenu**\
Budite u toku sa brzim svetom hakovanja kroz vesti i uvide u realnom vremenu

**Najnovije objave**\
Ostanite informisani o najnovijim pokretanjima nagrada za pronalaženje bagova i važnim ažuriranjima platforme

**Pridružite nam se na** [**Discord-u**](https://discord.com/invite/N3FrSbmwdy) i počnite da sarađujete sa vrhunskim hakerima danas!

### **Najbolji alat za pronalaženje vektora eskalacije privilegija na lokalnom Linuxu:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

### [Informacije o sistemu](privilege-escalation/#system-information)

* [ ] Dobijte **informacije o OS-u**
* [ ] Proverite [**PATH**](privilege-escalation/#path), bilo **koji pisivi folder**?
* [ ] Proverite [**env promenljive**](privilege-escalation/#env-info), bilo kakvi osetljivi detalji?
* [ ] Pretražite [**eksploate kernela**](privilege-escalation/#kernel-exploits) **koristeći skripte** (DirtyCow?)
* [ ] **Proverite** da li je [**verzija sudo-a ranjiva**](privilege-escalation/#sudo-version)
* [ ] [**Dmesg** verifikacija potpisa nije uspela](privilege-escalation/#dmesg-signature-verification-failed)
* [ ] Više sistema enum ([datum, statistika sistema, informacije o CPU-u, štampači](privilege-escalation/#more-system-enumeration))
* [ ] [Enumeracija dodatnih odbrana](privilege-escalation/#enumerate-possible-defenses)

### [Diskovi](privilege-escalation/#drives)

* [ ] **Izlistajte montirane** diskove
* [ ] **Postoji li neki nemonitrani disk**?
* [ ] **Ima li kredencijala u fstab-u**?

### [**Instalirani softver**](privilege-escalation/#installed-software)

* [ ] **Proverite da li je** [**instaliran koristan softver**](privilege-escalation/#useful-software)
* [ ] **Proverite da li je** [**instaliran ranjiv softver**](privilege-escalation/#vulnerable-software-installed)

### [Procesi](privilege-escalation/#processes)

* [ ] Da li se izvršava **nepoznat softver**?
* [ ] Da li se neki softver izvršava sa **više privilegija nego što bi trebalo**?
* [ ] Pretražite **eksploate pokrenutih procesa** (posebno verzija koja se izvršava).
* [ ] Možete li **izmeniti binarni fajl** nekog pokrenutog procesa?
* [ ] **Pratite procese** i proverite da li se često izvršava neki interesantan proces.
* [ ] Možete li **pročitati** neku interesantnu **memoriju procesa** (gde se mogu čuvati lozinke)?

### [Zakazani/Cron poslovi?](privilege-escalation/#scheduled-jobs)

* [ ] Da li je [**PATH** ](privilege-escalation/#cron-path)izmenjen od strane nekog cron-a i možete **pisati** u njega?
* [ ] Ima li [**zvezdica** ](privilege-escalation/#cron-using-a-script-with-a-wildcard-wildcard-injection)u cron poslu?
* [ ] Neki [**izmenjivi skript** ](privilege-escalation/#cron-script-overwriting-and-symlink)se **izvršava** ili se nalazi u **izmenjivom folderu**?
* [ ] Da li ste primetili da se neka **skripta** može ili se često **izvršava**]\(privilege-escalation/#frequent-cron-jobs)? (svakih 1, 2 ili 5 minuta)

### [Servisi](privilege-escalation/#services)

* [ ] Postoji li **pisiv .service** fajl?
* [ ] Postoji li **pisiv binarni fajl** koji se izvršava putem **servisa**?
* [ ] Postoji li **pisiv folder u systemd PATH-u**?

### [Tajmeri](privilege-escalation/#timers)

* [ ] Postoji li **pisiv tajmer**?

### [Soketi](privilege-escalation/#sockets)

* [ ] Postoji li **pisiv .socket** fajl?
* [ ] Možete li **komunicirati sa bilo kojim soketom**?
* [ ] **HTTP soketi** sa interesantnim informacijama?

### [D-Bus](privilege-escalation/#d-bus)

* [ ] Možete li **komunicirati sa bilo kojim D-Bus-om**?

### [Mreža](privilege-escalation/#network)

* [ ] Enumerišite mrežu da biste znali gde se nalazite
* [ ] **Otvoreni portovi kojima niste mogli pristupiti** pre nego što ste dobili shell unutar mašine?
* [ ] Možete li **snifovati saobraćaj** koristeći `tcpdump`?

### [Korisnici](privilege-escalation/#users)

* [ ] Opšte **enumerisanje korisnika/grupa**
* [ ] Imate li **vrlo veliki UID**? Da li je **mašina** **ranjiva**?
* [ ] Možete li [**povećati privilegije zahvaljujući grupi**](privilege-escalation/interesting-groups-linux-pe/) kojoj pripadate?
* [ ] Podaci iz **klipborda**?
* [ ] Politika loz

### [Mogućnosti](privilege-escalation/#capabilities)

* [ ] Da li bilo koji binarni fajl ima **neočekivanu mogućnost**?

### [ACL-ovi](privilege-escalation/#acls)

* [ ] Da li bilo koji fajl ima **neočekivani ACL**?

### [Otvorene sesije komandne linije](privilege-escalation/#open-shell-sessions)

* [ ] **screen**
* [ ] **tmux**

### [SSH](privilege-escalation/#ssh)

* [ ] **Debian** [**OpenSSL Predvidljivi PRNG - CVE-2008-0166**](privilege-escalation/#debian-openssl-predictable-prng-cve-2008-0166)
* [ ] [**Interesantne vrednosti konfiguracije SSH-a**](privilege-escalation/#ssh-interesting-configuration-values)

### [Interesantni fajlovi](privilege-escalation/#interesting-files)

* [ ] **Profilni fajlovi** - Čitanje osetljivih podataka? Pisanje za privesc?
* [ ] **passwd/shadow fajlovi** - Čitanje osetljivih podataka? Pisanje za privesc?
* [ ] **Provera često interesantnih foldera** za osetljive podatke
* [ ] **Čudna lokacija/vlasnički fajlovi**, možda imate pristup izvršnim fajlovima ili ih možete menjati
* [ ] **Izmenjeni** u poslednjih nekoliko minuta
* [ ] **Sqlite DB fajlovi**
* [ ] **Skriveni fajlovi**
* [ ] **Skripte/Binarni fajlovi u PATH-u**
* [ ] **Web fajlovi** (lozinke?)
* [ ] **Backup-ovi**?
* [ ] **Poznati fajlovi koji sadrže lozinke**: Koristite **Linpeas** i **LaZagne**
* [ ] **Opšte pretrage**

### [**Fajlovi sa dozvolom za pisanje**](privilege-escalation/#writable-files)

* [ ] **Izmena Python biblioteke** da bi se izvršile proizvoljne komande?
* [ ] Da li možete **izmeniti log fajlove**? Eksploatacija Logtotten-a
* [ ] Da li možete **izmeniti /etc/sysconfig/network-scripts/**? Eksploatacija na Centos/Redhat-u
* [ ] Da li možete [**pisati u ini, int.d, systemd ili rc.d fajlove**](privilege-escalation/#init-init-d-systemd-and-rc-d)?

### [**Drugi trikovi**](privilege-escalation/#other-tricks)

* [ ] Da li možete **zloupotrebiti NFS da biste dobili privilegije**]\(privilege-escalation/#nfs-privilege-escalation)?
* [ ] Da li trebate **izaći iz restriktivne ljuske**]\(privilege-escalation/#escaping-from-restricted-shells)?

<figure><img src="../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Pridružite se [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) serveru kako biste komunicirali sa iskusnim hakerima i lovcima na bagove!

**Hacking Insights**\
Uključite se u sadržaj koji istražuje uzbuđenje i izazove hakovanja

**Vesti o hakovanju u realnom vremenu**\
Budite u toku sa brzim svetom hakovanja kroz vesti i uvide u realnom vremenu

**Najnovije objave**\
Budite informisani o najnovijim pokretanjima bug bounty-ja i važnim ažuriranjima platforme

**Pridružite nam se na** [**Discord-u**](https://discord.com/invite/N3FrSbmwdy) i počnite da sarađujete sa vrhunskim hakerima danas!

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
