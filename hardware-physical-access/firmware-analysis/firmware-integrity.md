# Firmware Integrity

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### Integritet firmware-a

**Prilagođeni firmware i/ili kompajlirane binarne datoteke mogu biti otpremljene kako bi se iskoristile slabosti u integritetu ili proveri potpisa**. Sledeći koraci mogu biti praćeni za kompilaciju backdoor bind shell-a:

1. Firmware se može izvući pomoću firmware-mod-kit (FMK).
2. Treba identifikovati arhitekturu ciljnog firmware-a i endianness.
3. Može se izgraditi prelazni kompajler koristeći Buildroot ili druge odgovarajuće metode za okruženje.
4. Backdoor se može izgraditi koristeći prelazni kompajler.
5. Backdoor se može kopirati u izvučeni firmware /usr/bin direktorijum.
6. Odgovarajući QEMU binarni fajl se može kopirati u izvučeni firmware rootfs.
7. Backdoor se može emulirati koristeći chroot i QEMU.
8. Backdoor se može pristupiti putem netcat-a.
9. QEMU binarni fajl treba ukloniti iz izvučenog firmware rootfs-a.
10. Modifikovani firmware se može ponovo zapakovati koristeći FMK.
11. Backdoored firmware se može testirati emuliranjem sa alatom za analizu firmware-a (FAT) i povezivanjem sa ciljnom IP adresom i portom backdoor-a koristeći netcat.

Ako je već dobijena root shell putem dinamičke analize, manipulacije bootloader-a ili testiranja hardverske sigurnosti, mogu se izvršiti prekompajlirane zlonamerne binarne datoteke kao što su implantati ili reverse shell-ovi. Automatizovani alati za payload/implantate poput Metasploit framework-a i 'msfvenom' mogu se iskoristiti sledećim koracima:

1. Treba identifikovati arhitekturu ciljnog firmware-a i endianness.
2. Msfvenom se može koristiti za specificiranje ciljnog payload-a, IP adrese napadača, broja slušanja porta, tipa fajla, arhitekture, platforme i izlaznog fajla.
3. Payload se može preneti na kompromitovani uređaj i osigurati da ima dozvole za izvršavanje.
4. Metasploit se može pripremiti za obradu dolaznih zahteva pokretanjem msfconsole-a i konfigurisanjem podešavanja prema payload-u.
5. Meterpreter reverse shell se može izvršiti na kompromitovanom uređaju.
6. Meterpreter sesije se mogu pratiti kako se otvaraju.
7. Mogu se izvršiti post-exploitation aktivnosti.

Ako je moguće, slabosti unutar startup skripti mogu se iskoristiti kako bi se dobio trajni pristup uređaju tokom ponovnog pokretanja. Ove slabosti se javljaju kada startup skripte referenciraju, [simbolički linkuju](https://www.chromium.org/chromium-os/chromiumos-design-docs/hardening-against-malicious-stateful-data) ili zavise od koda koji se nalazi na nepouzdanim montiranim lokacijama poput SD kartica i fleš volumena koji se koriste za skladištenje podataka van root fajl sistema.

### Reference

* Za dodatne informacije pogledajte [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
