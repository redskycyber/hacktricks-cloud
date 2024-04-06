# Image Acquisition & Mount

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Akvizicija

### DD

```bash
#This will generate a raw copy of the disk
dd if=/dev/sdb of=disk.img
```

### dcfldd

dcfldd je napredni alat za kopiranje i konverziju slika. On pruža dodatne funkcionalnosti u odnosu na standardni dd alat, kao što su mogućnost prikaza napretka kopiranja, automatsko generisanje kontrolnih suma i mogućnost rada sa više izvora i odredišta istovremeno. Ovaj alat je veoma koristan prilikom akvizicije slika i kopiranja podataka sa oštećenih medija.

```bash
#Raw copy with hashes along the way (more secur as it checks hashes while it's copying the data)
dcfldd if=<subject device> of=<image file> bs=512 hash=<algorithm> hashwindow=<chunk size> hashlog=<hash file>
dcfldd if=/dev/sdc of=/media/usb/pc.image hash=sha256 hashwindow=1M hashlog=/media/usb/pc.hashes
```

### FTK Imager

Možete [**preuzeti FTK imager odavde**](https://accessdata.com/product-download/debian-and-ubuntu-x64-3-1-1).

```bash
ftkimager /dev/sdb evidence --e01 --case-number 1 --evidence-number 1 --description 'A description' --examiner 'Your name'
```

### EWF

Možete generisati sliku diska koristeći [**ewf alate**](https://github.com/libyal/libewf).

```bash
ewfacquire /dev/sdb
#Name: evidence
#Case number: 1
#Description: A description for the case
#Evidence number: 1
#Examiner Name: Your name
#Media type: fixed
#Media characteristics: physical
#File format: encase6
#Compression method: deflate
#Compression level: fast

#Then use default values
#It will generate the disk image in the current directory
```

## Montiranje

### Nekoliko vrsta

U **Windows**-u možete pokušati koristiti besplatnu verziju Arsenal Image Mounter-a ([https://arsenalrecon.com/downloads/](https://arsenalrecon.com/downloads/)) za **montiranje forenzičke slike**.

### Sirova

```bash
#Get file type
file evidence.img
evidence.img: Linux rev 1.0 ext4 filesystem data, UUID=1031571c-f398-4bfb-a414-b82b280cf299 (extents) (64bit) (large files) (huge files)

#Mount it
mount evidence.img /mnt
```

### EWF

EWF (EnCase Evidence File) je popularan format za snimanje slika dokaza. Ovaj format omogućava snimanje slike diska sa svim sektorima, uključujući i neiskorišćene sektore. EWF format takođe podržava kompresiju slike kako bi se smanjila veličina fajla.

Da biste izvršili akviziciju slike diska u EWF formatu, možete koristiti alate kao što su EnCase, FTK Imager ili ewfacquire. Ovi alati omogućavaju snimanje slike diska u EWF formatu sa svim relevantnim metapodacima.

Kada je slika diska snimljena u EWF formatu, možete je montirati kao virtualni disk kako biste pristupili podacima. Za montiranje EWF slike možete koristiti alate kao što su Arsenal Image Mounter, OSFMount ili FTK Imager.

Montiranje EWF slike omogućava vam pregledavanje i analizu podataka na disku bez potrebe za fizičkim pristupom originalnom disku. Ovo je korisno u forenzičkim istraživanjima, jer omogućava sigurno rukovanje dokazima i sprečava moguće oštećenje originalnog diska.

Važno je napomenuti da prilikom akvizicije slike diska u EWF formatu treba biti pažljiv kako bi se osigurala integritet dokaza. Takođe, treba voditi računa o pravilnom rukovanju i čuvanju EWF slike kako bi se očuvala njena autentičnost i integritet.

```bash
#Get file type
file evidence.E01
evidence.E01: EWF/Expert Witness/EnCase image file format

#Transform to raw
mkdir output
ewfmount evidence.E01 output/
file output/ewf1
output/ewf1: Linux rev 1.0 ext4 filesystem data, UUID=05acca66-d042-4ab2-9e9c-be813be09b24 (needs journal recovery) (extents) (64bit) (large files) (huge files)

#Mount
mount output/ewf1 -o ro,norecovery /mnt
```

### ArsenalImageMounter

To je Windows aplikacija za montiranje volumena. Možete je preuzeti ovde [https://arsenalrecon.com/downloads/](https://arsenalrecon.com/downloads/)

### Greške

* **`cannot mount /dev/loop0 read-only`** u ovom slučaju trebate koristiti zastavice **`-o ro,norecovery`**
* **`wrong fs type, bad option, bad superblock on /dev/loop0, missing codepage or helper program, or other error.`** u ovom slučaju montiranje nije uspelo jer je offset fajl sistema različit od offseta slike diska. Morate pronaći veličinu sektora i početni sektor:

```bash
fdisk -l disk.img
Disk disk.img: 102 MiB, 106954648 bytes, 208896 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00495395

Device        Boot Start    End Sectors  Size Id Type
disk.img1       2048 208895  206848  101M  1 FAT12
```

Imajte na umu da je veličina sektora **512**, a početak je **2048**. Zatim montirajte sliku na sledeći način:

```bash
mount disk.img /mnt -o ro,offset=$((2048*512))
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
