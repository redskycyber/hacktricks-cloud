# MISC

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite videti **oglašavanje vaše kompanije na HackTricks-u** ili **preuzeti HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

U ping odgovoru TTL:\
127 = Windows\
254 = Cisco\
Lo demás,algunlinux

$1$- md5\
$2$ili $2a$ - Blowfish\
$5$- sha256\
$6$- sha512

Ako ne znate šta se nalazi iza neke usluge, pokušajte da napravite HTTP GET zahtev.

**UDP Skeniranje**\
nc -nv -u -z -w 1 \<IP> 160-16

Prazan UDP paket se šalje na određeni port. Ako je UDP port otvoren, nema odgovora koji se vraća sa ciljne mašine. Ako je UDP port zatvoren, trebalo bi da se vrati ICMP paket sa porukom da je port nedostupan sa ciljne mašine.\\

UDP skeniranje portova često nije pouzdano, jer firewall-i i ruteri mogu odbaciti ICMP\
pakete. To može dovesti do lažnih pozitivnih rezultata u skeniranju, i često ćete videti\
da UDP skeniranje prikazuje sve UDP portove otvorene na skeniranoj mašini.\
o Većina port skenera ne skenira sve dostupne portove, već obično ima unapred definisanu listu\
"interesantnih portova" koje skenira.

## CTF - Trikovi

U **Windows-u** koristite **Winzip** za pretragu fajlova.\
**Alternate data Streams**: _dir /r | find ":$DATA"_\\

```
binwalk --dd=".*" <file> #Extract everything
binwalk -M -e -d=10000 suspicious.pdf #Extract, look inside extracted files and continue extracing (depth of 10000)
```

### Kripto

**featherduster**\\

**Basae64**(6—>8) —> 0...9, a...z, A…Z,+,/\
**Base32**(5 —>8) —> A…Z, 2…7\
**Base85** (Ascii85, 7—>8) —> 0...9, a...z, A...Z, ., -, :, +, =, ^, !, /, \*, ?, &, <, >, (, ), \[, ], {, }, @, %, $, #\
**Uuencode** --> Počinje sa "_begin \<mode> \<filename>_" i čudnim karakterima\
**Xxencoding** --> Počinje sa "_begin \<mode> \<filename>_" i B64\
\
**Vigenere** (analiza frekvencije) —> [https://www.guballa.de/vigenere-solver](https://www.guballa.de/vigenere-solver)\
**Scytale** (pomak karaktera) —> [https://www.dcode.fr/scytale-cipher](https://www.dcode.fr/scytale-cipher)

**25x25 = QR**

factordb.com\
rsatool

Snow --> Sakriva poruke koristeći razmake i tabove

## Karakteri

%E2%80%AE => RTL Karakter (payload piše unazad)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
