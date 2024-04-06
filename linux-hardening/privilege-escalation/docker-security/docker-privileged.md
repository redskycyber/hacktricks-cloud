# Docker --privileged

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Šta utiče

Kada pokrenete kontejner sa privilegijama, onemogućavate sledeće zaštite:

### Montiranje /dev

U privilegovanom kontejneru, svi **uređaji mogu biti pristupljeni u `/dev/`**. Stoga možete **izbeći** tako što ćete **montirati** disk domaćina.

{% tabs %}
{% tab title="Unutar podrazumevanog kontejnera" %}
```bash
# docker run --rm -it alpine sh
ls /dev
console  fd       mqueue   ptmx     random   stderr   stdout   urandom
core     full     null     pts      shm      stdin    tty      zero
```
{% endtab %}

{% tab title="Unutar privilegovanog kontejnera" %}
```bash
# docker run --rm --privileged -it alpine sh
ls /dev
cachefiles       mapper           port             shm              tty24            tty44            tty7
console          mem              psaux            stderr           tty25            tty45            tty8
core             mqueue           ptmx             stdin            tty26            tty46            tty9
cpu              nbd0             pts              stdout           tty27            tty47            ttyS0
[...]
```
{% endtab %}
{% endtabs %}

### Kernel fajl sistemi samo za čitanje

Kernel fajl sistemi pružaju mehanizam za proces da izmeni ponašanje kernela. Međutim, kada je reč o procesima kontejnera, želimo da sprečimo da izvrše bilo kakve promene na kernelu. Zato montiramo kernel fajl sisteme kao **samo za čitanje** unutar kontejnera, čime osiguravamo da procesi kontejnera ne mogu da menjaju kernel.

```bash
# docker run --rm -it alpine sh
mount | grep '(ro'
sysfs on /sys type sysfs (ro,nosuid,nodev,noexec,relatime)
cpuset on /sys/fs/cgroup/cpuset type cgroup (ro,nosuid,nodev,noexec,relatime,cpuset)
cpu on /sys/fs/cgroup/cpu type cgroup (ro,nosuid,nodev,noexec,relatime,cpu)
cpuacct on /sys/fs/cgroup/cpuacct type cgroup (ro,nosuid,nodev,noexec,relatime,cpuacct)
```

```bash
# docker run --rm --privileged -it alpine sh
mount  | grep '(ro'
```

### Maskiranje preko kernel fajl sistema

Fajl sistem **/proc** je selektivno upisiv, ali iz bezbednosnih razloga, određeni delovi su zaštićeni od upisa i čitanja preko preklapanja sa **tmpfs**, čime se osigurava da procesi kontejnera ne mogu pristupiti osetljivim područjima.

{% hint style="info" %}
**tmpfs** je fajl sistem koji čuva sve fajlove u virtuelnoj memoriji. tmpfs ne kreira fajlove na tvrdom disku. Dakle, ako demontirate tmpfs fajl sistem, svi fajlovi koji se u njemu nalaze su zauvek izgubljeni.
{% endhint %}

{% tabs %}
{% tab title="Unutar podrazumevanog kontejnera" %}
```bash
# docker run --rm -it alpine sh
mount  | grep /proc.*tmpfs
tmpfs on /proc/acpi type tmpfs (ro,relatime)
tmpfs on /proc/kcore type tmpfs (rw,nosuid,size=65536k,mode=755)
tmpfs on /proc/keys type tmpfs (rw,nosuid,size=65536k,mode=755)
```
{% endtab %}

{% tab title="Unutar privilegovanog kontejnera" %}
```bash
# docker run --rm --privileged -it alpine sh
mount  | grep /proc.*tmpfs
```
{% endtab %}
{% endtabs %}

### Linux sposobnosti

Pokretači kontejnera pokreću kontejnere sa **ograničenim brojem sposobnosti** kako bi kontrolisali šta se dešava unutar kontejnera prema podrazumevanim postavkama. **Privilegovani** kontejneri imaju **sve** **sposobnosti** dostupne. Da biste saznali više o sposobnostima, pročitajte:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="Unutar podrazumevanog kontejnera" %}
```bash
# docker run --rm -it alpine sh
apk add -U libcap; capsh --print
[...]
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=eip
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
[...]
```
{% endtab %}

{% tab title="Unutar privilegovanog kontejnera" %}
```bash
# docker run --rm --privileged -it alpine sh
apk add -U libcap; capsh --print
[...]
Current: =eip cap_perfmon,cap_bpf,cap_checkpoint_restore-eip
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
[...]
```
{% endtab %}
{% endtabs %}

Možete manipulisati sposobnostima dostupnim kontejneru bez pokretanja u `--privileged` režimu koristeći opcije `--cap-add` i `--cap-drop`.

### Seccomp

**Seccomp** je koristan za **ograničavanje** **sistemskih poziva** koje kontejner može izvršiti. Podrazumevani seccomp profil je omogućen podrazumevano prilikom pokretanja Docker kontejnera, ali je onemogućen u privilegovanom režimu. Saznajte više o Seccomp-u ovde:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="Unutar podrazumevanog kontejnera" %}
```bash
# docker run --rm -it alpine sh
grep Seccomp /proc/1/status
Seccomp:	2
Seccomp_filters:	1
```
{% endtab %}

{% tab title="Unutar privilegovanog kontejnera" %}
```bash
# docker run --rm --privileged -it alpine sh
grep Seccomp /proc/1/status
Seccomp:	0
Seccomp_filters:	0
```
{% endtab %}
{% endtabs %}

```bash
# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined
```

Takođe, napomena da kada se Docker (ili drugi CRIs) koristi u **Kubernetes** klasteru, **seccomp filter je podrazumevano onemogućen**.

### AppArmor

**AppArmor** je unapređenje jezgra za ograničavanje **kontejnera** na **ograničen skup resursa** sa **profilima po programu**. Kada pokrenete sa `--privileged` zastavicom, ova zaštita je onemogućena.

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

```bash
# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined
```

### SELinux

Pokretanje kontejnera sa `--privileged` zastavicom onemogućava **SELinux oznake**, što rezultira nasleđivanjem oznake kontejner motora, obično `unconfined`, što omogućava potpuni pristup sličan kontejner motoru. U režimu bez root prava, koristi se `container_runtime_t`, dok se u root režimu primenjuje `spc_t`.

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

```bash
# You can manually disable selinux in docker with
--security-opt label:disable
```

## Šta ne utiče

### Namespaces

Namespaces **NISU pogođeni** `--privileged` zastavicom. Iako nemaju omogućene sigurnosne restrikcije, **ne vide sve procese na sistemu ili mrežu domaćina, na primer**. Korisnici mogu onemogućiti pojedinačne namespaces koristeći **`--pid=host`, `--net=host`, `--ipc=host`, `--uts=host`** zastavice kontejnerskog motora.

{% tabs %}
{% tab title="Unutar podrazumevanog privilegovanog kontejnera" %}
```bash
# docker run --rm --privileged -it alpine sh
ps -ef
PID   USER     TIME  COMMAND
1 root      0:00 sh
18 root      0:00 ps -ef
```
{% endtab %}

{% tab title="Unutar --pid=host Kontejnera" %}
```bash
# docker run --rm --privileged --pid=host -it alpine sh
ps -ef
PID   USER     TIME  COMMAND
1 root      0:03 /sbin/init
2 root      0:00 [kthreadd]
3 root      0:00 [rcu_gp]ount | grep /proc.*tmpfs
[...]
```
{% endtab %}
{% endtabs %}

### Простор имен корисника

**По подразумеваном, контјенерски мотори не користе простор имена корисника, осим за контјенере без корена**, који их захтевају за монтирање фајл система и коришћење више УИД-ова. Простори имена корисника, који су неопходни за контјенере без корена, не могу бити онемогућени и значајно повећавају безбедност ограничавањем привилегија.

## Референце

* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)

<details>

<summary><strong>Научите хаковање AWS-а од нуле до хероја са</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Да ли радите у **компанији за кибер безбедност**? Желите ли да видите **вашу компанију рекламирану на HackTricks**? Или желите да имате приступ **најновијој верзији PEASS или преузмете HackTricks у PDF-у**? Проверите [**ПЛАНОВЕ ПРЕТПЛАТЕ**](https://github.com/sponsors/carlospolop)!
* Откријте [**The PEASS Family**](https://opensea.io/collection/the-peass-family), нашу колекцију ексклузивних [**NFT-ова**](https://opensea.io/collection/the-peass-family)
* Набавите [**званични PEASS & HackTricks сувенир**](https://peass.creator-spring.com)
* **Придружите се** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) или [**телеграм групи**](https://t.me/peass) или ме **пратите** на **Твитеру** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поделите своје хакерске трикове слањем PR-ова на** [**hacktricks репо**](https://github.com/carlospolop/hacktricks) **и** [**hacktricks-cloud репо**](https://github.com/carlospolop/hacktricks-cloud).

</details>
