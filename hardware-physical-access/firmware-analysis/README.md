# Firmware Analysis

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## **Uvod**

Firmware je osnovni softver koji omogućava uređajima pravilan rad upravljanjem i olakšavanjem komunikacije između hardverskih komponenti i softvera sa kojim korisnici interaguju. On se čuva u trajnoj memoriji, obezbeđujući da uređaj može pristupiti vitalnim instrukcijama od trenutka kada se uključi, što dovodi do pokretanja operativnog sistema. Ispitivanje i potencijalna modifikacija firmware-a je ključni korak u identifikaciji bezbednosnih ranjivosti.

## **Prikupljanje informacija**

**Prikupljanje informacija** je ključni početni korak u razumevanju sastava uređaja i tehnologija koje koristi. Ovaj proces uključuje prikupljanje podataka o:

* Arhitekturi CPU-a i operativnom sistemu koji se koristi
* Specifičnostima bootloader-a
* Hardverskom rasporedu i tehničkim listovima
* Metrikama koda i lokacijama izvora
* Spoljnim bibliotekama i vrstama licenci
* Istorijama ažuriranja i regulatornim sertifikatima
* Arhitektonskim i protokolarnim dijagramima
* Bezbednosnim procenama i identifikovanim ranjivostima

Za ove svrhe, alati za **otvorenu obaveštajnu (OSINT)** su neprocenjivi, kao i analiza dostupnih komponenti softvera otvorenog koda kroz ručne i automatske procese pregleda. Alati poput [Coverity Scan](https://scan.coverity.com) i [Semmle’s LGTM](https://lgtm.com/#explore) nude besplatnu statičku analizu koja se može iskoristiti za pronalaženje potencijalnih problema.

## **Dobijanje firmware-a**

Dobijanje firmware-a može se pristupiti na različite načine, pri čemu svaki ima svoj nivo složenosti:

* **Direktno** od izvora (programeri, proizvođači)
* **Izgradnja** prema pruženim instrukcijama
* **Preuzimanje** sa zvaničnih podrška sajtova
* Korišćenje **Google dork** upita za pronalaženje smeštenih firmware fajlova
* Direktan pristup **cloud skladištu**, uz pomoć alata kao što je [S3Scanner](https://github.com/sa7mon/S3Scanner)
* Presretanje **ažuriranja** putem tehnika man-in-the-middle
* **Izdvajanje** sa uređaja putem veza kao što su **UART**, **JTAG** ili **PICit**
* **Snifiranje** zahteva za ažuriranje unutar komunikacije uređaja
* Identifikacija i korišćenje **hardkodiranih krajnjih tačaka za ažuriranje**
* **Dumpovanje** iz bootloader-a ili mreže
* **Uklanjanje i čitanje** čipa za skladištenje, kada sve drugo ne uspe, koristeći odgovarajuće hardverske alate

## Analiza firmware-a

Sada kada **imate firmware**, trebate izvući informacije o njemu kako biste znali kako da ga obradite. Različiti alati koje možete koristiti za to:

```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #print offsets in hex
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head # might find signatures in header
fdisk -lu <bin> #lists a drives partition and filesystems if multiple
```

Ako ne pronađete mnogo pomoću tih alata, proverite **entropiju** slike pomoću `binwalk -E <bin>`. Ako je entropija niska, verovatno nije šifrovana. Ako je entropija visoka, verovatno je šifrovana (ili kompresovana na neki način).

Osim toga, možete koristiti ove alate za izdvajanje **fajlova ugrađenih u firmware**:

{% content-ref url="../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

Ili [**binvis.io**](https://binvis.io/#/) ([kod](https://code.google.com/archive/p/binvis/)) za pregled fajla.

### Dobijanje fajl sistema

Pomoću prethodno pomenutih alata kao što je `binwalk -ev <bin>`, trebali biste biti u mogućnosti da **izvučete fajl sistem**.\
Binwalk obično izvlači fajl sistem unutar **foldera koji se naziva po tipu fajl sistema**, a obično je jedan od sledećih: squashfs, ubifs, romfs, rootfs, jffs2, yaffs2, cramfs, initramfs.

#### Ručno izdvajanje fajl sistema

Ponekad, binwalk **nema magični bajt fajl sistema u svojim potpisima**. U tim slučajevima, koristite binwalk da **pronađete offset fajl sistema i izdvojite kompresovani fajl sistem** iz binarnog fajla i **ručno izdvojite** fajl sistem prema njegovom tipu koristeći sledeće korake.

```
$ binwalk DIR850L_REVB.bin

DECIMAL HEXADECIMAL DESCRIPTION
----------------------------------------------------------------------------- ---

0 0x0 DLOB firmware header, boot partition: """"dev=/dev/mtdblock/1""""
10380 0x288C LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 5213748 bytes
1704052 0x1A0074 PackImg section delimiter tag, little endian size: 32256 bytes; big endian size: 8257536 bytes
1704084 0x1A0094 Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 8256900 bytes, 2688 inodes, blocksize: 131072 bytes, created: 2016-07-12 02:28:41
```

Pokrenite sledeću **dd komandu** za izdvajanje Squashfs fajl sistema.

```
$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs

8257536+0 records in

8257536+0 records out

8257536 bytes (8.3 MB, 7.9 MiB) copied, 12.5777 s, 657 kB/s
```

Alternativno, mogla bi se pokrenuti i sledeća komanda.

`$ dd if=DIR850L_REVB.bin bs=1 skip=$((0x1A0094)) of=dir.squashfs`

* Za squashfs (korišćeno u prethodnom primeru)

`$ unsquashfs dir.squashfs`

Datoteke će se nalaziti u direktorijumu "`squashfs-root`" nakon toga.

* CPIO arhivske datoteke

`$ cpio -ivd --no-absolute-filenames -F <bin>`

* Za jffs2 fajl sisteme

`$ jefferson rootfsfile.jffs2`

* Za ubifs fajl sisteme sa NAND flešom

`$ ubireader_extract_images -u UBI -s <start_offset> <bin>`

`$ ubidump.py <bin>`

## Analiza Firmware-a

Kada se dobije firmware, neophodno je da se analizira kako bi se razumela njegova struktura i potencijalne ranjivosti. Ovaj proces uključuje korišćenje različitih alata za analizu i izvlačenje korisnih podataka iz slike firmware-a.

### Alati za početnu analizu

Niz komandi je dostupan za početni pregled binarnog fajla (nazvanog `<bin>`). Ove komande pomažu u identifikaciji vrsta datoteka, izvlačenju stringova, analizi binarnih podataka i razumevanju detalja particija i fajl sistema:

```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #prints offsets in hexadecimal
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head #useful for finding signatures in the header
fdisk -lu <bin> #lists partitions and filesystems, if there are multiple
```

Da biste procenili status enkripcije slike, proverava se **entropija** pomoću `binwalk -E <bin>`. Niska entropija ukazuje na nedostatak enkripcije, dok visoka entropija ukazuje na moguću enkripciju ili kompresiju.

Za izdvajanje **ugrađenih datoteka**, preporučuju se alati i resursi poput dokumentacije **file-data-carving-recovery-tools** i alata **binvis.io** za inspekciju datoteka.

### Izdvajanje fajl sistema

Korišćenjem `binwalk -ev <bin>`, obično se može izdvojiti fajl sistem, često u direktorijum sa nazivom fajl sistemskog tipa (npr. squashfs, ubifs). Međutim, kada **binwalk** ne uspe da prepozna tip fajl sistema zbog nedostajućih magičnih bajtova, neophodno je ručno izdvajanje. To uključuje korišćenje `binwalk`-a za pronalaženje offseta fajl sistema, a zatim korišćenje `dd` komande za izdvajanje fajl sistema:

```bash
$ binwalk DIR850L_REVB.bin

$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs
```

Nakon toga, u zavisnosti od tipa fajl sistema (npr. squashfs, cpio, jffs2, ubifs), koriste se različite komande za ručno izvlačenje sadržaja.

### Analiza fajl sistema

Nakon izvlačenja fajl sistema, započinje se pretraga sigurnosnih propusta. Posebna pažnja se posvećuje nesigurnim mrežnim demonima, unapred definisanim akreditivima, API endpointima, funkcionalnostima servera za ažuriranje, nekompilovanom kodu, skriptama za pokretanje i kompilovanim binarnim fajlovima za offline analizu.

**Ključne lokacije** i **elementi** koje treba pregledati uključuju:

* **etc/shadow** i **etc/passwd** za korisničke akreditive
* SSL sertifikate i ključeve u **etc/ssl**
* Konfiguracione i skript fajlove za potencijalne ranjivosti
* Ugrađene binarne fajlove za dalju analizu
* Uobičajene veb servere i binarne fajlove IoT uređaja

Nekoliko alata pomaže u otkrivanju osetljivih informacija i ranjivosti u fajl sistemu:

* [**LinPEAS**](https://github.com/carlospolop/PEASS-ng) i [**Firmwalker**](https://github.com/craigz28/firmwalker) za pretragu osetljivih informacija
* [**The Firmware Analysis and Comparison Tool (FACT)**](https://github.com/fkie-cad/FACT\_core) za sveobuhvatnu analizu firmware-a
* [**FwAnalyzer**](https://github.com/cruise-automation/fwanalyzer), [**ByteSweep**](https://gitlab.com/bytesweep/bytesweep), [**ByteSweep-go**](https://gitlab.com/bytesweep/bytesweep-go) i [**EMBA**](https://github.com/e-m-b-a/emba) za statičku i dinamičku analizu

### Provere sigurnosti na kompilovanim binarnim fajlovima

Izvorni kod i kompilovani binarni fajlovi pronađeni u fajl sistemu moraju biti pažljivo pregledani radi otkrivanja ranjivosti. Alati poput **checksec.sh** za Unix binarne fajlove i **PESecurity** za Windows binarne fajlove pomažu u identifikaciji nezaštićenih binarnih fajlova koji mogu biti iskorišćeni.

## Emulacija firmware-a za dinamičku analizu

Proces emulacije firmware-a omogućava **dinamičku analizu** ili rada uređaja ili pojedinačnog programa. Ovaj pristup može naići na izazove sa hardverom ili zavisnostima od arhitekture, ali prenos fajl sistema korenskog direktorijuma ili određenih binarnih fajlova na uređaj sa odgovarajućom arhitekturom i endianess-om, poput Raspberry Pi-a, ili na prethodno izgrađenu virtuelnu mašinu, može olakšati dalje testiranje.

### Emulacija pojedinačnih binarnih fajlova

Za ispitivanje pojedinačnih programa, ključno je identifikovati endianess i arhitekturu programa.

#### Primer sa MIPS arhitekturom

Za emulaciju binarnog fajla sa MIPS arhitekturom, može se koristiti komanda:

```bash
file ./squashfs-root/bin/busybox
```

I da biste instalirali potrebne alate za emulaciju:

```bash
sudo apt-get install qemu qemu-user qemu-user-static qemu-system-arm qemu-system-mips qemu-system-x86 qemu-utils
```

Za MIPS (big-endian), koristi se `qemu-mips`, a za little-endian binarne datoteke, izbor bi bio `qemu-mipsel`.

#### Emulacija ARM arhitekture

Za ARM binarne datoteke, proces je sličan, pri čemu se koristi emulator `qemu-arm` za emulaciju.

### Emulacija celog sistema

Alati poput [Firmadyne](https://github.com/firmadyne/firmadyne), [Firmware Analysis Toolkit](https://github.com/attify/firmware-analysis-toolkit) i drugi olakšavaju potpunu emulaciju firmware-a, automatizujući proces i pomažući u dinamičkoj analizi.

## Praktična analiza u pokretu

U ovoj fazi koristi se stvarno ili emulirano okruženje uređaja za analizu. Važno je održavati pristup ljusci operativnog sistema i fajl sistemu. Emulacija možda neće savršeno oponašati interakcije sa hardverom, pa će povremeno biti potrebno ponovno pokretanje emulacije. Analiza treba ponovo pregledati fajl sistem, iskoristiti izložene web stranice i mrežne servise, istražiti ranjivosti bootloader-a. Testiranje integriteta firmware-a je ključno za identifikaciju potencijalnih ranjivosti zadnjih vrata.

## Tehnike analize u toku izvršavanja

Analiza u toku izvršavanja uključuje interakciju sa procesom ili binarnom datotekom u njenom operativnom okruženju, koristeći alate poput gdb-multiarch, Frida i Ghidra za postavljanje prekida i identifikaciju ranjivosti putem fuzzinga i drugih tehnika.

## Eksploatacija binarnih datoteka i dokaz koncepta

Razvoj dokaza koncepta za identifikovane ranjivosti zahteva duboko razumevanje ciljne arhitekture i programiranje u jezicima nižeg nivoa. Zaštite binarnih datoteka u ugrađenim sistemima su retke, ali kada su prisutne, mogu biti neophodne tehnike poput Return Oriented Programming (ROP).

## Pripremljeni operativni sistemi za analizu firmware-a

Operativni sistemi poput [AttifyOS](https://github.com/adi0x90/attifyos) i [EmbedOS](https://github.com/scriptingxss/EmbedOS) pružaju prekonfigurirana okruženja za testiranje sigurnosti firmware-a, opremljena neophodnim alatima.

## Pripremljeni operativni sistemi za analizu firmware-a

* [**AttifyOS**](https://github.com/adi0x90/attifyos): AttifyOS je distribucija namenjena pomoći pri oceni sigurnosti i testiranju prodiranja uređaja Internet of Things (IoT). Uštedeće vam puno vremena pružajući prekonfigurirano okruženje sa svim neophodnim alatima.
* [**EmbedOS**](https://github.com/scriptingxss/EmbedOS): Operativni sistem za testiranje sigurnosti ugrađenih sistema zasnovan na Ubuntu 18.04 sa prethodno učitanim alatima za testiranje sigurnosti firmware-a.

## Ranjivi firmware-i za vežbanje

Za vežbanje otkrivanja ranjivosti u firmware-u, koristite sledeće projekte ranjivih firmware-a kao polaznu tačku.

* OWASP IoTGoat
* [https://github.com/OWASP/IoTGoat](https://github.com/OWASP/IoTGoat)
* The Damn Vulnerable Router Firmware Project
* [https://github.com/praetorian-code/DVRF](https://github.com/praetorian-code/DVRF)
* Damn Vulnerable ARM Router (DVAR)
* [https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html](https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html)
* ARM-X
* [https://github.com/therealsaumil/armx#downloads](https://github.com/therealsaumil/armx#downloads)
* Azeria Labs VM 2.0
* [https://azeria-labs.com/lab-vm-2-0/](https://azeria-labs.com/lab-vm-2-0/)
* Damn Vulnerable IoT Device (DVID)
* [https://github.com/Vulcainreo/DVID](https://github.com/Vulcainreo/DVID)

## Reference

* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)
* [Practical IoT Hacking: The Definitive Guide to Attacking the Internet of Things](https://www.amazon.co.uk/Practical-IoT-Hacking-F-Chantzis/dp/1718500904)

## Obuka i sertifikacija

* [https://www.attify-store.com/products/offensive-iot-exploitation](https://www.attify-store.com/products/offensive-iot-exploitation)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
