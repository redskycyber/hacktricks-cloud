# Unpacking binaries

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Identifikacija pakovanih binarnih fajlova

* **Nedostatak stringova**: Često se dešava da pakovani binarni fajlovi nemaju gotovo nijedan string.
* Mnogo **neiskorišćenih stringova**: Takođe, kada malver koristi neku vrstu komercijalnog pakera, često se nalazi mnogo stringova bez prekoračenja. Čak i ako ti stringovi postoje, to ne znači da binarni fajl nije pakovan.
* Takođe možete koristiti neke alate da biste pokušali da pronađete koji je paker korišćen za pakovanje binarnog fajla:
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

## Osnovne preporuke

* **Počnite** analizirajući pakovani binarni fajl **odozdo u IDA-i i krenite prema gore**. Unpackeri izlaze kada izlazi raspakovani kod, pa je malo verovatno da će unpacker preneti izvršenje na raspakovani kod na početku.
* Tražite **JMP-ove** ili **CALL-ove** ka **registrima** ili **regionima** memorije. Takođe tražite **funkcije koje guraju argumente i adresu pravca, a zatim pozivaju `retn`**, jer povratak funkcije u tom slučaju može pozvati adresu koja je upravo stavljena na stek pre poziva.
* Stavite **prekidnu tačku** na `VirtualAlloc` jer ovo alocira prostor u memoriji gde program može pisati raspakovani kod. Pokrenite do korisničkog koda ili koristite F8 da biste **dobili vrednost unutar EAX registra** nakon izvršenja funkcije i "**pratite tu adresu u dump-u**". Nikad ne znate da li je to region gde će se sačuvati raspakovani kod.
* **`VirtualAlloc`** sa vrednošću "**40**" kao argument znači Read+Write+Execute (ovde će biti kopiran kod koji zahteva izvršavanje).
* Dok raspakujete kod, normalno je da pronađete **nekoliko poziva** aritmetičkih operacija i funkcija poput **`memcopy`** ili **`Virtual`**`Alloc`. Ako se nađete u funkciji koja očigledno samo vrši aritmetičke operacije i možda neki `memcopy`, preporuka je da pokušate da **pronađete kraj funkcije** (možda JMP ili poziv nekom registru) **ili** barem **poziv poslednje funkcije** i pokrenete se do nje jer kod nije interesantan.
* Dok raspakujete kod, **zabeležite** svaki put kada **promenite region memorije**, jer promena regiona memorije može ukazivati na **početak raspakovnog koda**. Možete lako dump-ovati region memorije koristeći Process Hacker (process --> properties --> memory).
* Dok pokušavate da raspakujete kod, dobar način da **znate da li već radite sa raspakovanim kodom** (tako da ga možete samo dump-ovati) je da **proverite stringove binarnog fajla**. Ako u nekom trenutku izvršite skok (možda promena regiona memorije) i primetite da je **dodato mnogo više stringova**, onda možete znati da **radite sa raspakovanim kodom**.\
  Međutim, ako paket već sadrži mnogo stringova, možete videti koliko stringova sadrži reč "http" i videti da li se taj broj povećava.
* Kada dump-ujete izvršni fajl iz regiona memorije, možete popraviti neke zaglavlja koristeći [PE-bear](https://github.com/hasherezade/pe-bear-releases/releases).

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
