# Seccomp

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

**Seccomp**, što znači Secure Computing mode, je sigurnosna funkcija **Linux kernela koja filtrira sistemski pozive**. On ograničava procese na ograničeni skup sistemskih poziva (`exit()`, `sigreturn()`, `read()` i `write()`) za već otvorene file deskriptore. Ako proces pokuša da pozove bilo šta drugo, kernel ga prekida korišćenjem SIGKILL ili SIGSYS signala. Ovaj mehanizam ne virtualizuje resurse, već ih izoluje od procesa.

Postoje dva načina za aktiviranje seccomp-a: kroz `prctl(2)` sistemski poziv sa `PR_SET_SECCOMP`, ili za Linux kernel verzije 3.17 i novije, kroz `seccomp(2)` sistemski poziv. Stariji način omogućavanja seccomp-a pisanjem u `/proc/self/seccomp` je zastareo u korist `prctl()`.

Unapređenje, **seccomp-bpf**, dodaje mogućnost filtriranja sistemskih poziva sa prilagodljivom politikom, koristeći Berkeley Packet Filter (BPF) pravila. Ovo proširenje se koristi u softverima kao što su OpenSSH, vsftpd i Chrome/Chromium pregledači na Chrome OS-u i Linux-u za fleksibilno i efikasno filtriranje sistemskih poziva, nudeći alternativu za sada nepodržani systrace za Linux.

### **Originalni/Striktni režim**

U ovom režimu Seccomp **samo dozvoljava sistemski pozive** `exit()`, `sigreturn()`, `read()` i `write()` za već otvorene file deskriptore. Ako se izvrši bilo koji drugi sistemski poziv, proces se ubija korišćenjem SIGKILL signala.

{% code title="seccomp_strict.c" %}
```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <linux/seccomp.h>
#include <sys/prctl.h>

//From https://sysdig.com/blog/selinux-seccomp-falco-technical-discussion/
//gcc seccomp_strict.c -o seccomp_strict

int main(int argc, char **argv)
{
int output = open("output.txt", O_WRONLY);
const char *val = "test";

//enables strict seccomp mode
printf("Calling prctl() to set seccomp strict mode...\n");
prctl(PR_SET_SECCOMP, SECCOMP_MODE_STRICT);

//This is allowed as the file was already opened
printf("Writing to an already open file...\n");
write(output, val, strlen(val)+1);

//This isn't allowed
printf("Trying to open file for reading...\n");
int input = open("output.txt", O_RDONLY);

printf("You will not see this message--the process will be killed first\n");
}
```
{% endcode %}

### Seccomp-bpf

Ovaj režim omogućava **filtriranje sistemskih poziva pomoću konfigurabilne politike** implementirane pomoću pravila Berkeley Packet Filter.

{% code title="seccomp_bpf.c" %}
```c
#include <seccomp.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

//https://security.stackexchange.com/questions/168452/how-is-sandboxing-implemented/175373
//gcc seccomp_bpf.c -o seccomp_bpf -lseccomp

void main(void) {
/* initialize the libseccomp context */
scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_KILL);

/* allow exiting */
printf("Adding rule : Allow exit_group\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);

/* allow getting the current pid */
//printf("Adding rule : Allow getpid\n");
//seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(getpid), 0);

printf("Adding rule : Deny getpid\n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(getpid), 0);
/* allow changing data segment size, as required by glibc */
printf("Adding rule : Allow brk\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(brk), 0);

/* allow writing up to 512 bytes to fd 1 */
printf("Adding rule : Allow write upto 512 bytes to FD 1\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 2,
SCMP_A0(SCMP_CMP_EQ, 1),
SCMP_A2(SCMP_CMP_LE, 512));

/* if writing to any other fd, return -EBADF */
printf("Adding rule : Deny write to any FD except 1 \n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(write), 1,
SCMP_A0(SCMP_CMP_NE, 1));

/* load and enforce the filters */
printf("Load rules and enforce \n");
seccomp_load(ctx);
seccomp_release(ctx);
//Get the getpid is denied, a weird number will be returned like
//this process is -9
printf("this process is %d\n", getpid());
}
```
{% endcode %}

## Seccomp u Dockeru

**Seccomp-bpf** je podržan od strane **Dockera** kako bi se ograničili **syscalls** iz kontejnera, efektivno smanjujući površinu napada. Možete pronaći **blokirane syscalls** po **podrazumevanim postavkama** na [https://docs.docker.com/engine/security/seccomp/](https://docs.docker.com/engine/security/seccomp/) i **podrazumevani seccomp profil** možete pronaći ovde [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json).\
Možete pokrenuti Docker kontejner sa **različitom seccomp** politikom koristeći:

```bash
docker run --rm \
-it \
--security-opt seccomp=/path/to/seccomp/profile.json \
hello-world
```

Ako na primer želite da **zabranite** kontejneru da izvršava neke **syscall**-ove kao što je `uname`, možete preuzeti podrazumevani profil sa [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) i jednostavno **ukloniti string `uname` iz liste**.\
Ako želite da se uverite da **neki binarni fajl ne radi unutar docker kontejnera**, možete koristiti strace da biste izlistali syscall-ove koje binarni fajl koristi, a zatim ih zabraniti.\
U sledećem primeru su otkriveni **syscall**-ovi za `uname`:

```bash
docker run -it --security-opt seccomp=default.json modified-ubuntu strace uname
```

{% hint style="info" %}
Ako koristite **Docker samo za pokretanje aplikacije**, možete je **profilisati** pomoću **`strace`** i **samo dozvoliti syscalls** koje aplikacija zahteva.
{% endhint %}

### Primer Seccomp politike

[Primer sa ovog linka](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)

Da bismo ilustrovali Seccomp funkcionalnost, kreirajmo Seccomp profil koji onemogućava "chmod" sistemski poziv kako je prikazano ispod.

```json
{
"defaultAction": "SCMP_ACT_ALLOW",
"syscalls": [
{
"name": "chmod",
"action": "SCMP_ACT_ERRNO"
}
]
}
```

U prethodnom profilu smo postavili podrazumevanu akciju na "dozvoli" i napravili crnu listu da onemogućimo "chmod". Da bismo bili sigurniji, možemo postaviti podrazumevanu akciju na "odbaci" i napraviti belu listu da selektivno omogućimo sistemski poziv.\
Sledeći izlaz prikazuje da poziv "chmod" vraća grešku jer je onemogućen u seccomp profilu.

```bash
$ docker run --rm -it --security-opt seccomp:/home/smakam14/seccomp/profile.json busybox chmod 400 /etc/hosts
chmod: /etc/hosts: Operation not permitted
```

Sledeći izlaz prikazuje "docker inspect" koji prikazuje profil:

```json
"SecurityOpt": [
"seccomp:{\"defaultAction\":\"SCMP_ACT_ALLOW\",\"syscalls\":[{\"name\":\"chmod\",\"action\":\"SCMP_ACT_ERRNO\"}]}"
],
```

### Isključivanje u Dockeru

Pokrenite kontejner sa zastavicom: **`--security-opt seccomp=unconfined`**

Od verzije Kubernetes 1.19, **seccomp je podrazumevano omogućen za sve Podove**. Međutim, podrazumevani seccomp profil koji se primenjuje na Podove je profil "**RuntimeDefault**", koji je **pružen od strane kontejner runtime-a** (npr. Docker, containerd). Profil "RuntimeDefault" dozvoljava većinu sistemskih poziva, ali blokira nekoliko koji se smatraju opasnim ili općenito nisu potrebni za kontejnere.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
