# DDexec / EverythingExec

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Kontekst

U Linux-u, da bi se pokrenuo program, on mora postojati kao fajl, mora biti dostupan na neki način kroz hijerarhiju fajl sistema (ovo je samo kako `execve()` funkcija radi). Taj fajl može biti smešten na disku ili u ramu (tmpfs, memfd), ali vam je potreban putanja do fajla. Ovo je olakšalo kontrolu onoga što se pokreće na Linux sistemu, olakšava otkrivanje pretnji i alata napadača ili sprečavanje njihovog pokušaja izvršavanja bilo čega od njih (_npr._ ne dozvoljavajući neprivilegovanim korisnicima da postavljaju izvršne fajlove bilo gde).

Ali ova tehnika menja sve to. Ako ne možete pokrenuti željeni proces... **onda preuzimate već postojeći**.

Ova tehnika vam omogućava da **zaobiđete uobičajene tehnike zaštite kao što su samo čitanje, zabrana izvršavanja, bela lista imena fajlova, bela lista heševa...**

## Zavisnosti

Konačni skript zavisi od sledećih alata da bi radio, oni moraju biti dostupni na sistemu koji napadate (podrazumevano ćete ih svuda pronaći):

```
dd
bash | zsh | ash (busybox)
head
tail
cut
grep
od
readlink
wc
tr
base64
```

## Tehnika

Ako možete proizvoljno izmeniti memoriju procesa, možete ga preuzeti. Ovo se može koristiti za preuzimanje već postojećeg procesa i zamenjivanje drugim programom. To možemo postići ili korišćenjem `ptrace()` sistemskog poziva (koji zahteva mogućnost izvršavanja sistemskih poziva ili prisustvo gdb-a na sistemu) ili, što je interesantnije, pisanjem u `/proc/$pid/mem`.

Datoteka `/proc/$pid/mem` je jedan-na-jedan mapiranje celokupnog adresnog prostora procesa (_npr._ od `0x0000000000000000` do `0x7ffffffffffff000` u x86-64). To znači da čitanje ili pisanje u ovu datoteku na offsetu `x` isto je kao čitanje ili izmena sadržaja na virtuelnoj adresi `x`.

Sada, imamo četiri osnovna problema sa kojima se suočavamo:

* Opšte uzev, samo root i vlasnik programa mogu ga izmeniti.
* ASLR.
* Ako pokušamo čitati ili pisati na adresu koja nije mapirana u adresnom prostoru programa, dobićemo I/O grešku.

Ovi problemi imaju rešenja koja, iako nisu savršena, su dobra:

* Većina shell interpretera omogućava kreiranje file deskriptora koji će biti nasleđeni od strane child procesa. Možemo kreirati fd koji pokazuje na `mem` datoteku školjke sa dozvolama za pisanje... tako da će child procesi koji koriste taj fd moći da izmene memoriju školjke.
* ASLR čak nije ni problem, možemo proveriti `maps` datoteku školjke ili bilo koju drugu iz procfs-a kako bismo dobili informacije o adresnom prostoru procesa.
* Dakle, moramo `lseek()` preko datoteke. Iz školjke to ne može biti urađeno osim korišćenjem zloglasnog `dd`.

### Detaljnije

Koraci su relativno jednostavni i ne zahtevaju nikakvo stručno znanje da biste ih razumeli:

* Analizirajte binarni fajl koji želimo da pokrenemo i loader kako biste saznali koja mapiranja im je potrebno. Zatim kreirajte "shell" kod koji će izvršiti, općenito govoreći, iste korake koje kernel obavlja pri svakom pozivu `execve()`:
* Kreirajte ta mapiranja.
* Učitajte binarne fajlove u njih.
* Podesite dozvole.
* Na kraju, inicijalizujte stek sa argumentima za program i postavite pomoćni vektor (potreban od strane loadera).
* Skočite u loader i pustite ga da obavi ostatak (učitavanje biblioteka potrebnih za program).
* Dobijte iz fajla `syscall` adresu na koju će se proces vratiti nakon izvršavanja sistemskog poziva.
* Prepišite to mesto, koje će biti izvršivo, sa našim shell kodom (preko `mem` možemo izmeniti stranice koje nisu za pisanje).
* Prosledite program koji želite pokrenuti na stdin procesa (će biti `read()` od strane pomenutog "shell" koda).
* U ovom trenutku je na loaderu da učita potrebne biblioteke za naš program i skoči u njega.

**Pogledajte alat na** [**https://github.com/arget13/DDexec**](https://github.com/arget13/DDexec)

## EverythingExec

Postoji nekoliko alternativa za `dd`, od kojih je jedna, `tail`, trenutno podrazumevani program koji se koristi za `lseek()` kroz `mem` datoteku (što je bio jedini razlog za korišćenje `dd`). Pomenute alternative su:

```bash
tail
hexdump
cmp
xxd
```

Postavljanjem promenljive `SEEKER` možete promeniti korišćeni seeker, npr.:

```bash
SEEKER=cmp bash ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```

Ako pronađete još jedan važeći seeker koji nije implementiran u skriptu, i dalje ga možete koristiti postavljanjem promenljive `SEEKER_ARGS`:

```bash
SEEKER=xxd SEEKER_ARGS='-s $offset' zsh ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```

Blokiraj ovo, EDR-ovi.

## Reference

* [https://github.com/arget13/DDexec](https://github.com/arget13/DDexec)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
