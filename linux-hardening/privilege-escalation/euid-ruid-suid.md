# euid, ruid, suid

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

### Promenljive za identifikaciju korisnika

* **`ruid`**: **Real user ID** označava korisnika koji je pokrenuo proces.
* **`euid`**: Poznat kao **efektivni user ID**, predstavlja korisnički identitet koji sistem koristi da bi utvrdio privilegije procesa. Uglavnom, `euid` odražava `ruid`, osim u slučajevima izvršavanja SetUID binarnih fajlova, gde `euid` preuzima identitet vlasnika fajla, čime se dodeljuju određene operativne dozvole.
* **`suid`**: Ovaj **sačuvani user ID** je ključan kada visoko privilegovan proces (obično pokrenut kao root) privremeno mora da odustane od svojih privilegija radi obavljanja određenih zadataka, da bi kasnije povratio svoj početni povišeni status.

#### Važna napomena

Proces koji ne radi pod root-om može samo da izmeni svoj `euid` da se podudara sa trenutnim `ruid`, `euid` ili `suid`.

### Razumevanje set\*uid funkcija

* **`setuid`**: Za razliku od prvobitnih pretpostavki, `setuid` pretežno menja `euid`, a ne `ruid`. Konkretno, za privilegovane procese, usklađuje `ruid`, `euid` i `suid` sa određenim korisnikom, često root-om, čime se efektivno utvrđuju ovi ID-ovi zbog preklapanja `suid`. Detaljnije informacije mogu se pronaći na [setuid man stranici](https://man7.org/linux/man-pages/man2/setuid.2.html).
* **`setreuid`** i **`setresuid`**: Ove funkcije omogućavaju nijansirano podešavanje `ruid`, `euid` i `suid`. Međutim, njihove mogućnosti zavise od nivoa privilegija procesa. Za procese koji nisu root, izmene su ograničene na trenutne vrednosti `ruid`, `euid` i `suid`. Nasuprot tome, root procesi ili oni sa `CAP_SETUID` mogu dodeliti proizvoljne vrednosti ovim ID-ovima. Više informacija se može pronaći na [setresuid man stranici](https://man7.org/linux/man-pages/man2/setresuid.2.html) i [setreuid man stranici](https://man7.org/linux/man-pages/man2/setreuid.2.html).

Ove funkcionalnosti su dizajnirane ne kao mehanizam za bezbednost, već da olakšaju željeni operativni tok, kao što je kada program preuzima identitet drugog korisnika menjanjem svog efektivnog user ID-a.

Važno je napomenuti da, iako je `setuid` često korišćen za povišenje privilegija na root (jer usklađuje sve ID-ove sa root-om), razlikovanje između ovih funkcija je ključno za razumevanje i manipulaciju ponašanjem user ID-ova u različitim scenarijima.

### Mekanizmi izvršavanja programa u Linux-u

#### **`execve` sistemski poziv**

* **Funkcionalnost**: `execve` pokreće program koji je određen prvom argumentom. Koristi dva niza argumenata, `argv` za argumente i `envp` za okruženje.
* **Ponašanje**: Zadržava memorijski prostor pozivaoca, ali osvežava stek, hip i segmente podataka. Kod programa se zamenjuje novim programom.
* **Očuvanje korisničkog ID-a**:
* `ruid`, `euid` i dodatni grupni ID-ovi ostaju nepromenjeni.
* `euid` može imati nijansirane promene ako novi program ima postavljen SetUID bit.
* `suid` se ažurira iz `euid` nakon izvršenja.
* **Dokumentacija**: Detaljne informacije mogu se pronaći na [`execve` man stranici](https://man7.org/linux/man-pages/man2/execve.2.html).

#### **`system` funkcija**

* **Funkcionalnost**: Za razliku od `execve`, `system` kreira dete proces koristeći `fork` i izvršava komandu unutar tog dete procesa koristeći `execl`.
* **Izvršavanje komande**: Izvršava komandu putem `sh` sa `execl("/bin/sh", "sh", "-c", command, (char *) NULL);`.
* **Ponašanje**: Budući da je `execl` oblik `execve`, funkcioniše slično, ali u kontekstu novog dete procesa.
* **Dokumentacija**: Dodatne informacije se mogu dobiti sa [`system` man stranice](https://man7.org/linux/man-pages/man3/system.3.html).

#### **Ponašanje `bash`-a i `sh`-a sa SUID**

* **`bash`**:
* Ima opciju `-p` koja utiče na to kako se tretiraju `euid` i `ruid`.
* Bez `-p`, `bash` postavlja `euid` na `ruid` ako se razlikuju na početku.
* Sa `-p`, početni `euid` se čuva.
* Više detalja se može pronaći na [`bash` man stranici](https://linux.die.net/man/1/bash).
* **`sh`**:
* Nema mehanizam sličan `-p` u `bash`-u.
* Ponašanje u vezi sa korisničkim ID-ovima nije eksplicitno navedeno, osim pod opcijom `-i`, koja naglašava očuvanje jednakosti `euid` i `ruid`.
* Dodatne informacije su dostupne na [`sh` man stranici](https://man7.org/linux/man-pages/man1/sh.1p.html).

Ovi mehanizmi, različiti u svom radu, pružaju raznovrsne opcije za izvršavanje i prelazak između programa, sa specifičnim nijansama u upravljanju i očuvanju korisničkih ID-ova.

### Testiranje ponašanja korisničkih ID-ova pri izvršavanju

Primeri preuzeti sa https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail, proverite za dodatne informacije

#### Slučaj 1: Korišćenje `setuid` sa `system`

**Cilj**: Razumevanje efekta `setuid` u kombinaciji sa `system` i `bash` kao `sh`.

**C kod**:

```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
system("id");
return 0;
}
```

**Kompilacija i dozvole:**

Kada kompilirate izvorni kod, važno je obratiti pažnju na dozvole datoteka. Ako izvršna datoteka ima postavljenu setuid (suid) dozvolu, ona će se izvršavati sa privilegijama vlasnika datoteke, umesto sa privilegijama korisnika koji je pokrenuo program. Ovo može biti korisno za izvršavanje određenih operacija koje zahtevaju privilegije koje korisnik nema.

Da biste postavili suid dozvolu na izvršnu datoteku, koristite komandu `chmod u+s ime_datoteke`. Da biste uklonili suid dozvolu, koristite komandu `chmod u-s ime_datoteke`.

Važno je napomenuti da je suid dozvola potencijalna sigurnosna rupa. Ako se izvršna datoteka sa suid dozvolom može zloupotrebiti, napadač može dobiti privilegije vlasnika datoteke i izvršavati neovlaštene operacije.

Da biste pronašli izvršne datoteke sa suid dozvolom, možete koristiti komandu `find / -perm -4000 -type f 2>/dev/null`. Ova komanda će pretražiti sistem i prikazati sve izvršne datoteke sa suid dozvolom.

Kada pronađete izvršnu datoteku sa suid dozvolom, važno je pažljivo proveriti da li postoji bilo kakva sigurnosna rupa koja bi mogla biti iskorišćena. Ako pronađete takvu ranjivost, obavestite odgovorne strane kako bi se problem rešio.

```bash
oxdf@hacky$ gcc a.c -o /mnt/nfsshare/a;
oxdf@hacky$ chmod 4755 /mnt/nfsshare/a
```

```bash
bash-4.2$ $ ./a
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```

**Analiza:**

* `ruid` i `euid` počinju kao 99 (nobody) i 1000 (frank) redom.
* `setuid` poravnava oba na 1000.
* `system` izvršava `/bin/bash -c id` zbog simboličke veze između sh i bash.
* `bash`, bez `-p`, prilagođava `euid` da se podudara sa `ruid`, rezultirajući da oba budu 99 (nobody).

#### Slučaj 2: Korišćenje setreuid sa sistemom

**C kod**:

```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setreuid(1000, 1000);
system("id");
return 0;
}
```

**Kompilacija i dozvole:**

Kada kompilirate izvorni kod, važno je obratiti pažnju na dozvole datoteka. Ako izvršna datoteka ima postavljenu setuid (suid) dozvolu, ona će se izvršavati sa privilegijama vlasnika datoteke, umesto sa privilegijama korisnika koji je pokrenuo program. Ovo može biti korisno za izvršavanje određenih operacija koje zahtevaju privilegije koje korisnik nema.

Da biste postavili suid dozvolu na izvršnu datoteku, koristite komandu `chmod u+s ime_datoteke`. Da biste uklonili suid dozvolu, koristite komandu `chmod u-s ime_datoteke`.

Važno je napomenuti da je suid dozvola potencijalna sigurnosna rupa. Ako se izvršna datoteka sa suid dozvolom može zloupotrebiti, napadač može dobiti privilegije vlasnika datoteke i izvršavati neovlaštene operacije.

Da biste pronašli izvršne datoteke sa suid dozvolom, možete koristiti komandu `find / -perm -4000 -type f 2>/dev/null`. Ova komanda će pretražiti sistem i prikazati sve izvršne datoteke sa suid dozvolom.

Kada pronađete izvršnu datoteku sa suid dozvolom, važno je pažljivo proveriti da li postoji bilo kakva sigurnosna rupa koja bi mogla biti iskorišćena. Ako pronađete takvu ranjivost, obavestite odgovorne strane kako bi se problem rešio.

```bash
oxdf@hacky$ gcc b.c -o /mnt/nfsshare/b; chmod 4755 /mnt/nfsshare/b
```

**Izvršenje i rezultat:**

```bash
bash-4.2$ $ ./b
uid=1000(frank) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```

**Analiza:**

* `setreuid` postavlja i ruid i euid na 1000.
* `system` poziva bash, koji održava identifikatore korisnika zbog njihove jednakosti, efektivno delujući kao frank.

#### Slučaj 3: Korišćenje setuid sa execve

Cilj: Istraživanje interakcije između setuid i execve.

```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/usr/bin/id", NULL, NULL);
return 0;
}
```

**Izvršenje i rezultat:**

```bash
bash-4.2$ $ ./c
uid=99(nobody) gid=99(nobody) euid=1000(frank) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```

**Analiza:**

* `ruid` ostaje 99, ali `euid` je postavljen na 1000, u skladu sa efektom `setuid`-a.

**C Primer koda 2 (Pozivanje Bash-a):**

```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/bin/bash", NULL, NULL);
return 0;
}
```

**Izvršenje i rezultat:**

```bash
bash-4.2$ $ ./d
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```

**Analiza:**

* Iako je `euid` postavljen na 1000 pomoću `setuid`, `bash` resetuje euid na `ruid` (99) zbog odsustva opcije `-p`.

**C Primer Koda 3 (Korišćenje bash -p):**

```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
char *const paramList[10] = {"/bin/bash", "-p", NULL};
setuid(1000);
execve(paramList[0], paramList, NULL);
return 0;
}
```

**Izvršenje i rezultat:**

```bash
bash-4.2$ $ ./e
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) euid=100
```

## Reference

* [https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail](https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
