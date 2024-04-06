# Linux Forensics

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da biste lako izgradili i **automatizovali radne tokove** pokretane najnaprednijim alatima zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Inicijalno prikupljanje informacija

### Osnovne informacije

Pre svega, preporučuje se da imate neki **USB** sa **poznatim binarnim fajlovima i bibliotekama** (možete jednostavno uzeti Ubuntu i kopirati foldere _/bin_, _/sbin_, _/lib,_ i _/lib64_), zatim montirajte USB, i izmenite okružne promenljive da biste koristili te binarne fajlove:

```bash
export PATH=/mnt/usb/bin:/mnt/usb/sbin
export LD_LIBRARY_PATH=/mnt/usb/lib:/mnt/usb/lib64
```

Kada ste konfigurisali sistem da koristi dobre i poznate binarne datoteke, možete početi **izvlačiti osnovne informacije**:

```bash
date #Date and time (Clock may be skewed, Might be at a different timezone)
uname -a #OS info
ifconfig -a || ip a #Network interfaces (promiscuous mode?)
ps -ef #Running processes
netstat -anp #Proccess and ports
lsof -V #Open files
netstat -rn; route #Routing table
df; mount #Free space and mounted devices
free #Meam and swap space
w #Who is connected
last -Faiwx #Logins
lsmod #What is loaded
cat /etc/passwd #Unexpected data?
cat /etc/shadow #Unexpected data?
find /directory -type f -mtime -1 -print #Find modified files during the last minute in the directory
```

#### Sumnjive informacije

Prilikom dobijanja osnovnih informacija trebalo bi proveriti čudne stvari kao što su:

* **Root procesi** obično se izvršavaju sa niskim PID-ovima, pa ako pronađete root proces sa velikim PID-om, možete posumnjati
* Proverite **registrovane prijave** korisnika bez ljuske unutar `/etc/passwd`
* Proverite **hash-ove lozinki** unutar `/etc/shadow` za korisnike bez ljuske

### Damp memorije

Za dobijanje memorije pokrenutog sistema, preporučuje se korišćenje [**LiME**](https://github.com/504ensicsLabs/LiME).\
Da biste ga **kompajlirali**, morate koristiti **isti kernel** koji koristi mašina žrtva.

{% hint style="info" %}
Zapamtite da **ne možete instalirati LiME ili bilo šta drugo** na mašini žrtve jer će to izvršiti nekoliko promena na njoj
{% endhint %}

Dakle, ako imate identičnu verziju Ubuntua, možete koristiti `apt-get install lime-forensics-dkms`\
U drugim slučajevima, morate preuzeti [**LiME**](https://github.com/504ensicsLabs/LiME) sa github-a i kompajlirati ga sa odgovarajućim kernel headerima. Da biste **dobili tačne kernel headere** mašine žrtve, jednostavno **kopirajte direktorijum** `/lib/modules/<kernel verzija>` na svoju mašinu, a zatim **kompajlirajte** LiME koristeći ih:

```bash
make -C /lib/modules/<kernel version>/build M=$PWD
sudo insmod lime.ko "path=/home/sansforensics/Desktop/mem_dump.bin format=lime"
```

LiME podržava 3 **formata**:

* Sirovi (svaki segment je konkateniran zajedno)
* Padded (isti kao sirovi, ali sa nulama na desnoj strani)
* Lime (preporučeni format sa metapodacima)

LiME takođe može biti korišćen za **slanje ispisa putem mreže** umesto čuvanja na sistemu korišćenjem nečega poput: `path=tcp:4444`

### Snimanje diska

#### Gašenje

Prvo, moraćete **ugasiti sistem**. Ovo nije uvek opcija jer će nekada sistem biti serverski sistem koji kompanija ne može da si priušti da isključi.\
Postoje **2 načina** gašenja sistema, **normalno gašenje** i **gašenje "izvadi utikač"**. Prvi će dozvoliti **procesima da se završe kao i obično** i **datotečnom sistemu** da se **sinhronizuje**, ali će takođe dozvoliti mogućem **malveru** da **uništi dokaze**. Pristup "izvadi utikač" može doneti **gubitak nekih informacija** (neće biti izgubljeno mnogo informacija jer smo već napravili sliku memorije) i **malver neće imati priliku** da bilo šta uradi povodom toga. Stoga, ako **sumnjate** da postoji **malver**, jednostavno izvršite **`sync`** **komandu** na sistemu i izvucite utikač.

#### Pravljenje slike diska

Važno je napomenuti da **pre nego što povežete svoj računar sa bilo čim što je povezano sa slučajem**, morate biti sigurni da će biti **montiran samo za čitanje** kako biste izbegli menjanje bilo kakvih informacija.

```bash
#Create a raw copy of the disk
dd if=<subject device> of=<image file> bs=512

#Raw copy with hashes along the way (more secure as it checks hashes while it's copying the data)
dcfldd if=<subject device> of=<image file> bs=512 hash=<algorithm> hashwindow=<chunk size> hashlog=<hash file>
dcfldd if=/dev/sdc of=/media/usb/pc.image hash=sha256 hashwindow=1M hashlog=/media/usb/pc.hashes
```

### Predanaliza slike diska

Kreiranje slike diska bez dodatnih podataka.

```bash
#Find out if it's a disk image using "file" command
file disk.img
disk.img: Linux rev 1.0 ext4 filesystem data, UUID=59e7a736-9c90-4fab-ae35-1d6a28e5de27 (extents) (64bit) (large files) (huge files)

#Check which type of disk image it's
img_stat -t evidence.img
raw
#You can list supported types with
img_stat -i list
Supported image format types:
raw (Single or split raw file (dd))
aff (Advanced Forensic Format)
afd (AFF Multiple File)
afm (AFF with external metadata)
afflib (All AFFLIB image formats (including beta ones))
ewf (Expert Witness Format (EnCase))

#Data of the image
fsstat -i raw -f ext4 disk.img
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: Ext4
Volume Name:
Volume ID: 162850f203fd75afab4f1e4736a7e776

Last Written at: 2020-02-06 06:22:48 (UTC)
Last Checked at: 2020-02-06 06:15:09 (UTC)

Last Mounted at: 2020-02-06 06:15:18 (UTC)
Unmounted properly
Last mounted on: /mnt/disk0

Source OS: Linux
[...]

#ls inside the image
fls -i raw -f ext4 disk.img
d/d 11: lost+found
d/d 12: Documents
d/d 8193:       folder1
d/d 8194:       folder2
V/V 65537:      $OrphanFiles

#ls inside folder
fls -i raw -f ext4 disk.img 12
r/r 16: secret.txt

#cat file inside image
icat -i raw -f ext4 disk.img 16
ThisisTheMasterSecret
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da biste lako izgradili i **automatizovali radne tokove** pokretane najnaprednijim alatima zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Pretraga poznatih Malvera

### Modifikovane sistemskih datoteka

Linux nudi alate za osiguravanje integriteta sistemskih komponenti, ključnih za otkrivanje potencijalno problematičnih datoteka.

* **Sistemi zasnovani na RedHat-u**: Koristite `rpm -Va` za sveobuhvatnu proveru.
* **Sistemi zasnovani na Debian-u**: `dpkg --verify` za početnu verifikaciju, zatim `debsums | grep -v "OK$"` (nakon instaliranja `debsums` sa `apt-get install debsums`) da identifikujete bilo kakve probleme.

### Detektori Malvera/Rootkita

Pročitajte sledeću stranicu da biste saznali o alatima koji mogu biti korisni za pronalaženje malvera:

{% content-ref url="malware-analysis.md" %}
[malware-analysis.md](malware-analysis.md)
{% endcontent-ref %}

## Pretraga instaliranih programa

Da biste efikasno pretraživali instalirane programe na Debian i RedHat sistemima, razmotrite korišćenje sistema logova i baza podataka zajedno sa ručnim proverama u uobičajenim direktorijumima.

* Za Debian, pregledajte _**`/var/lib/dpkg/status`**_ i _**`/var/log/dpkg.log`**_ da biste dobili detalje o instalacijama paketa, koristeći `grep` za filtriranje specifičnih informacija.
* Korisnici RedHat-a mogu upitati RPM bazu podataka sa `rpm -qa --root=/mntpath/var/lib/rpm` da bi dobili listu instaliranih paketa.

Da biste otkrili softver instaliran ručno ili van ovih upravljača paketima, istražite direktorijume poput _**`/usr/local`**_, _**`/opt`**_, _**`/usr/sbin`**_, _**`/usr/bin`**_, _**`/bin`**_, i _**`/sbin`**_. Kombinujte listu direktorijuma sa sistemskim komandama kako biste identifikovali izvršne datoteke koje nisu povezane sa poznatim paketima, unapređujući tako pretragu svih instaliranih programa.

```bash
# Debian package and log details
cat /var/lib/dpkg/status | grep -E "Package:|Status:"
cat /var/log/dpkg.log | grep installed
# RedHat RPM database query
rpm -qa --root=/mntpath/var/lib/rpm
# Listing directories for manual installations
ls /usr/sbin /usr/bin /bin /sbin
# Identifying non-package executables (Debian)
find /sbin/ -exec dpkg -S {} \; | grep "no path found"
# Identifying non-package executables (RedHat)
find /sbin/ –exec rpm -qf {} \; | grep "is not"
# Find exacuable files
find / -type f -executable | grep <something>
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da biste lako izgradili i **automatizovali radne tokove** pokretane najnaprednijim alatima zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Obnovite obrisane pokrenute binarne datoteke

Zamislite proces koji je izvršen iz /tmp/exec a zatim obrisan. Moguće je izvući ga

```bash
cd /proc/3746/ #PID with the exec file deleted
head -1 maps #Get address of the file. It was 08048000-08049000
dd if=mem bs=1 skip=08048000 count=1000 of=/tmp/exec2 #Recorver it
```

## Pregled lokacija automatskog pokretanja

### Planirani zadaci

```bash
cat /var/spool/cron/crontabs/*  \
/var/spool/cron/atjobs \
/var/spool/anacron \
/etc/cron* \
/etc/at* \
/etc/anacrontab \
/etc/incron.d/* \
/var/spool/incron/* \

#MacOS
ls -l /usr/lib/cron/tabs/ /Library/LaunchAgents/ /Library/LaunchDaemons/ ~/Library/LaunchAgents/
```

### Servisi

Putanje gde se zlonamerni softver može instalirati kao servis:

* **/etc/inittab**: Poziva inicijalne skripte poput rc.sysinit, usmeravajući dalje ka skriptama za pokretanje.
* **/etc/rc.d/** i **/etc/rc.boot/**: Sadrže skripte za pokretanje servisa, pri čemu se potonje nalazi u starijim verzijama Linuxa.
* **/etc/init.d/**: Koristi se u određenim verzijama Linuxa poput Debiana za čuvanje skripti za pokretanje.
* Servisi se takođe mogu aktivirati putem **/etc/inetd.conf** ili **/etc/xinetd/**, u zavisnosti od varijante Linuxa.
* **/etc/systemd/system**: Direktorijum za sistemske i upravljačke skripte servisa.
* **/etc/systemd/system/multi-user.target.wants/**: Sadrži linkove ka servisima koji treba da se pokrenu u multi-korisničkom nivou.
* **/usr/local/etc/rc.d/**: Za prilagođene ili servise trećih strana.
* **\~/.config/autostart/**: Za aplikacije koje se automatski pokreću specifične za korisnika, što može biti skrovište za zlonamerni softver usmeren ka korisniku.
* **/lib/systemd/system/**: Podrazumevane sistemski fajlovi jedinica koje pružaju instalirani paketi.

### Kernel Moduli

Linux kernel moduli, često korišćeni od strane zlonamernog softvera kao komponente rootkita, učitavaju se prilikom pokretanja sistema. Direktorijumi i fajlovi od suštinskog značaja za ove module uključuju:

* **/lib/modules/$(uname -r)**: Čuva module za pokrenutu verziju kernela.
* **/etc/modprobe.d**: Sadrži konfiguracione fajlove za kontrolu učitavanja modula.
* **/etc/modprobe** i **/etc/modprobe.conf**: Fajlovi za globalna podešavanja modula.

### Ostale Lokacije za Automatsko Pokretanje

Linux koristi različite fajlove za automatsko izvršavanje programa prilikom prijave korisnika, potencijalno skrivajući zlonamerni softver:

* **/etc/profile.d/**\*, **/etc/profile**, i **/etc/bash.bashrc**: Izvršavaju se prilikom prijave bilo kog korisnika.
* **\~/.bashrc**, **\~/.bash\_profile**, **\~/.profile**, i **\~/.config/autostart**: Fajlovi specifični za korisnika koji se pokreću prilikom njihove prijave.
* **/etc/rc.local**: Pokreće se nakon što su svi sistemske servisi pokrenuti, označavajući kraj tranzicije ka multi-korisničkom okruženju.

## Pregledajte Logove

Linux sistemi prate aktivnosti korisnika i događaje sistema putem različitih log fajlova. Ovi logovi su ključni za identifikaciju neovlašćenog pristupa, infekcija zlonamernim softverom i drugih sigurnosnih incidenata. Ključni log fajlovi uključuju:

* **/var/log/syslog** (Debian) ili **/var/log/messages** (RedHat): Snimaju sistemske poruke i aktivnosti na nivou sistema.
* **/var/log/auth.log** (Debian) ili **/var/log/secure** (RedHat): Beleže pokušaje autentifikacije, uspešne i neuspešne prijave.
* Koristite `grep -iE "session opened for|accepted password|new session|not in sudoers" /var/log/auth.log` za filtriranje relevantnih autentifikacionih događaja.
* **/var/log/boot.log**: Sadrži poruke o pokretanju sistema.
* **/var/log/maillog** ili **/var/log/mail.log**: Beleže aktivnosti email servera, korisne za praćenje servisa povezanih sa email-om.
* **/var/log/kern.log**: Čuva kernel poruke, uključujući greške i upozorenja.
* **/var/log/dmesg**: Drži poruke drajvera uređaja.
* **/var/log/faillog**: Beleži neuspele pokušaje prijave, pomažući u istraživanju sigurnosnih incidenata.
* **/var/log/cron**: Beleži izvršavanja cron poslova.
* **/var/log/daemon.log**: Prati aktivnosti pozadinskih servisa.
* **/var/log/btmp**: Dokumentuje neuspele pokušaje prijave.
* **/var/log/httpd/**: Sadrži Apache HTTPD greške i pristupne logove.
* **/var/log/mysqld.log** ili **/var/log/mysql.log**: Beleže aktivnosti MySQL baze podataka.
* **/var/log/xferlog**: Zabeležava FTP prenose fajlova.
* **/var/log/**: Uvek proverite ovde za neočekivane logove.

{% hint style="info" %}
Linux sistemi logove i podsisteme za reviziju mogu biti isključeni ili obrisani u slučaju upada ili zlonamernog softvera. Pošto logovi na Linux sistemima generalno sadrže neke od najkorisnijih informacija o zlonamernim aktivnostima, napadači ih rutinski brišu. Stoga, prilikom pregleda dostupnih log fajlova, važno je tražiti praznine ili nepravilne unose koji bi mogli biti indikacija brisanja ili manipulacije.
{% endhint %}

**Linux čuva istoriju komandi za svakog korisnika**, smeštenu u:

* \~/.bash\_history
* \~/.zsh\_history
* \~/.zsh\_sessions/\*
* \~/.python\_history
* \~/.\*\_history

Takođe, komanda `last -Faiwx` pruža listu korisničkih prijava. Proverite je za nepoznate ili neočekivane prijave.

Proverite fajlove koji mogu dati dodatne privilegije:

* Pregledajte `/etc/sudoers` za neočekivane korisničke privilegije koje su možda dodeljene.
* Pregledajte `/etc/sudoers.d/` za neočekivane korisničke privilegije koje su možda dodeljene.
* Ispitajte `/etc/groups` kako biste identifikovali bilo kakvo neobično članstvo u grupama ili dozvole.
* Ispitajte `/etc/passwd` kako biste identifikovali bilo kakvo neobično članstvo u grupama ili dozvole.

Neki programi takođe generišu svoje logove:

* **SSH**: Ispitajte _\~/.ssh/authorized\_keys_ i _\~/.ssh/known\_hosts_ za neovlašćene udaljene konekcije.
* **Gnome Desktop**: Pogledajte _\~/.recently-used.xbel_ za nedavno pristupane fajlove putem Gnome aplikacija.
* **Firefox/Chrome**: Proverite istoriju pretrage i preuzimanja u _\~/.mozilla/firefox_ ili _\~/.config/google-chrome_ za sumnjive aktivnosti.
* **VIM**: Pregledajte _\~/.viminfo_ za detalje o korišćenju, poput putanja do pristupljenih fajlova i istorije pretrage.
* **Open Office**: Proverite nedavni pristup dokumentima koji mogu ukazivati na kompromitovane fajlove.
* **FTP/SFTP**: Pregledajte logove u _\~/.ftp\_history_ ili _\~/.sftp\_history_ za prenose fajlova koji bi mogli biti neovlašćeni.
* **MySQL**: Istražite _\~/.mysql\_history_ za izvršene MySQL upite, što može otkriti neovlaštene aktivnosti u bazi podataka.
* **Less**: Analizirajte _\~/.lesshst_ za istoriju korišćenja, uključujući pregledane fajlove i izvršene komande.
* **Git**: Ispitajte _\~/.gitconfig_ i projekat _.git/logs_ za promene u repozitorijumima.

### USB Logovi

[**usbrip**](https://github.com/snovvcrash/usbrip) je mali softver napisan u čistom Pythonu 3 koji parsira Linux log fajlove (`/var/log/syslog*` ili `/var/log/messages*` u zavisnosti od distribucije) radi konstrukcije tabela istorije USB događaja.

Važno je **znati sve USB uređaje koji su korišćeni** i biće korisno ako imate autorizovanu listu USB uređaja kako biste pronašli "događaje kršenja" (korišćenje USB uređaja koji nisu na toj listi).

### Instalacija

```bash
pip3 install usbrip
usbrip ids download #Download USB ID database
```

### Primeri

```bash
usbrip events history #Get USB history of your curent linux machine
usbrip events history --pid 0002 --vid 0e0f --user kali #Search by pid OR vid OR user
#Search for vid and/or pid
usbrip ids download #Downlaod database
usbrip ids search --pid 0002 --vid 0e0f #Search for pid AND vid
```

Više primera i informacija možete pronaći na github-u: [https://github.com/snovvcrash/usbrip](https://github.com/snovvcrash/usbrip)

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako kreirate i **automatizujete radne tokove** podržane najnaprednijim alatima zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Pregled Korisničkih Računa i Aktivnosti Prijavljivanja

Ispitajte _**/etc/passwd**_, _**/etc/shadow**_ i **bezbednosne dnevnike** za neobična imena ili naloge kreirane i/ili korišćene u blizini poznatih neovlašćenih događaja. Takođe, proverite moguće brute-force napade na sudo.\
Takođe, proverite datoteke poput _**/etc/sudoers**_ i _**/etc/groups**_ za neočekivane privilegije date korisnicima.\
Na kraju, potražite naloge bez **šifri** ili sa **lako pogodnim** šiframa.

## Ispitivanje Fajl Sistema

### Analiza Struktura Fajl Sistema u Istraživanju Malvera

Prilikom istraživanja incidenata sa malverom, struktura fajl sistema je ključni izvor informacija, otkrivajući kako se događaji odvijaju i sadržaj malvera. Međutim, autori malvera razvijaju tehnike koje otežavaju ovu analizu, poput menjanja vremena datoteka ili izbegavanja fajl sistema za skladištenje podataka.

Da biste se suprotstavili ovim anti-forenzičkim metodama, bitno je:

* **Sprovesti temeljnu analizu vremenske linije** koristeći alate poput **Autopsy** za vizualizaciju vremenskih linija događaja ili **Sleuth Kit's** `mactime` za detaljne podatke o vremenskoj liniji.
* **Istražiti neočekivane skripte** u $PATH sistemu, koje mogu uključivati skripte ljuske ili PHP skripte korišćene od strane napadača.
* **Ispitati `/dev` za atipične fajlove**, pošto tradicionalno sadrži specijalne fajlove, ali može sadržati fajlove povezane sa malverom.
* **Tražiti skrivene fajlove ili direktorijume** sa imenima poput ".. " (tačka tačka razmak) ili "..^G" (tačka tačka kontrola-G), koji bi mogli sakriti zlonamerni sadržaj.
* **Identifikovati setuid root fajlove** korišćenjem komande: `find / -user root -perm -04000 -print` Ovo pronalazi fajlove sa povišenim dozvolama, koje napadači mogu zloupotrebiti.
* **Pregledati vremena brisanja** u inode tabelama kako biste primetili masovna brisanja fajlova, što može ukazivati na prisustvo rootkita ili trojanaca.
* **Ispitati uzastopne inode** za bliske zlonamerne fajlove nakon identifikacije jednog, jer su možda postavljeni zajedno.
* **Proveriti zajedničke binarne direktorijume** (_/bin_, _/sbin_) za nedavno modifikovane fajlove, jer bi ovi fajlovi mogli biti promenjeni od strane malvera.

````bash
# List recent files in a directory:
ls -laR --sort=time /bin```

# Sort files in a directory by inode:
ls -lai /bin | sort -n```
````

{% hint style="info" %}
Imajte na umu da **napadač** može **izmeniti** **vreme** da bi **datoteke izgledale** **legitimno**, ali ne može da izmeni **inode**. Ako otkrijete da **datoteka** pokazuje da je kreirana i izmenjena u **istom vremenu** kao i ostale datoteke u istom folderu, ali je **inode** **neočekivano veći**, onda su **vremenske oznake te datoteke izmenjene**.
{% endhint %}

## Uporedite datoteke različitih verzija fajl sistema

### Sažetak Uporedbe Verzija Fajl Sistema

Za upoređivanje verzija fajl sistema i precizno lociranje promena, koristimo pojednostavljene `git diff` komande:

* **Da biste pronašli nove datoteke**, uporedite dva direktorijuma:

```bash
git diff --no-index --diff-filter=A path/to/old_version/ path/to/new_version/
```

* **Za izmenjen sadržaj**, navedite promene ignorišući specifične linije:

```bash
git diff --no-index --diff-filter=M path/to/old_version/ path/to/new_version/ | grep -E "^\+" | grep -v "Installed-Time"
```

* **Za otkrivanje izbrisanih fajlova**:

```bash
git diff --no-index --diff-filter=D path/to/old_version/ path/to/new_version/
```

* **Opcije filtera** (`--diff-filter`) pomažu u sužavanju na specifične promene poput dodatih (`A`), izbrisanih (`D`) ili izmenjenih (`M`) fajlova.
* `A`: Dodati fajlovi
* `C`: Kopirani fajlovi
* `D`: Izbrisani fajlovi
* `M`: Izmenjeni fajlovi
* `R`: Preimenovani fajlovi
* `T`: Promene tipa (npr. fajl u simboličku vezu)
* `U`: Nespajani fajlovi
* `X`: Nepoznati fajlovi
* `B`: Oštećeni fajlovi

## Reference

* [https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf](https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf)
* [https://www.plesk.com/blog/featured/linux-logs-explained/](https://www.plesk.com/blog/featured/linux-logs-explained/)
* [https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203)
* **Knjiga: Malware Forensics Field Guide for Linux Systems: Digital Forensics Field Guides**

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Da li radite u **kompaniji za sajber bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF-u**? Proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!

* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) **Discord grupi**]\(https://discord.gg/hRep4RUj7f) ili **telegram grupi** ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**

**Podelite svoje hakovanje trikove slanjem PR-ova** [**hacktricks repozitorijumu**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijumu**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako izgradite i **automatizujete radne tokove** pokretane najnaprednijim alatima zajednice.\
Dobijte pristup danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
