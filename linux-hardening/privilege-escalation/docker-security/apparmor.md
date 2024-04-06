# AppArmor

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovne informacije

AppArmor je **kernel unapređenje dizajnirano da ograniči resurse dostupne programima putem profila za svaki program**, efektivno implementirajući obaveznu kontrolu pristupa (MAC) vezivanjem atributa kontrole pristupa direktno za programe umesto za korisnike. Ovaj sistem funkcioniše tako što **učitava profile u kernel**, obično tokom pokretanja, a ovi profili određuju koje resurse program može da pristupi, kao što su mrežne veze, pristup sirovim soketima i dozvole za fajlove.

Postoje dva operativna moda za AppArmor profile:

* **Mod za sprovođenje**: Ovaj mod aktivno sprovodi politike definisane unutar profila, blokirajući akcije koje krše ove politike i beležeći svaki pokušaj njihovog kršenja putem sistema kao što su syslog ili auditd.
* **Mod za pritužbe**: Za razliku od moda za sprovođenje, mod za pritužbe ne blokira akcije koje idu protiv politika profila. Umesto toga, beleži ove pokušaje kao kršenja politika bez sprovođenja ograničenja.

### Komponente AppArmor-a

* **Kernel modul**: Odgovoran za sprovođenje politika.
* **Politike**: Određuju pravila i ograničenja za ponašanje programa i pristup resursima.
* **Parser**: Učitava politike u kernel radi sprovođenja ili izveštavanja.
* **Alati**: Ovo su programi u korisničkom režimu koji pružaju interfejs za interakciju i upravljanje AppArmor-om.

### Putanje profila

AppArmor profili se obično čuvaju u _**/etc/apparmor.d/**_\
Sa `sudo aa-status` možete da izlistate binarne datoteke koje su ograničene nekim profilom. Ako možete da zamenite znak "/" tačkom u putanji svake navedene binarne datoteke, dobićete ime apparmor profila unutar pomenutog foldera.

Na primer, **apparmor** profil za _/usr/bin/man_ će se nalaziti u _/etc/apparmor.d/usr.bin.man_

### Komande

```bash
aa-status     #check the current status
aa-enforce    #set profile to enforce mode (from disable or complain)
aa-complain   #set profile to complain mode (from diable or enforcement)
apparmor_parser #to load/reload an altered policy
aa-genprof    #generate a new profile
aa-logprof    #used to change the policy when the binary/program is changed
aa-mergeprof  #used to merge the policies
```

## Kreiranje profila

* Da biste označili pogođeni izvršni fajl, dozvoljeni su **apsolutni putovi i džokere** (za globiranje fajlova) za specificiranje fajlova.
* Da biste označili pristup koji će binarni fajl imati nad **fajlovima**, mogu se koristiti sledeće **kontrole pristupa**:
* **r** (čitanje)
* **w** (pisanje)
* **m** (mapiranje u memoriju kao izvršni fajl)
* **k** (zaključavanje fajlova)
* **l** (kreiranje hard linkova)
* **ix** (izvršavanje drugog programa sa novim programom koji nasleđuje politiku)
* **Px** (izvršavanje pod drugim profilom, nakon čišćenja okruženja)
* **Cx** (izvršavanje pod dečjim profilom, nakon čišćenja okruženja)
* **Ux** (izvršavanje bez ograničenja, nakon čišćenja okruženja)
* **Promenljive** mogu biti definisane u profilima i mogu se manipulisati izvan profila. Na primer: @{PROC} i @{HOME} (dodajte #include \<tunables/global> u fajl profila)
* **Pravila zabrane su podržana za prevođenje pravila dozvole**.

### aa-genprof

Da biste lako započeli kreiranje profila, apparmor vam može pomoći. Moguće je da **apparmor pregleda akcije koje izvršni fajl izvršava, a zatim vam omogući da odlučite koje akcije želite da dozvolite ili zabranite**.\
Samo trebate pokrenuti:

```bash
sudo aa-genprof /path/to/binary
```

Zatim, u drugoj konzoli izvršite sve radnje koje će binarna datoteka obično izvršiti:

```bash
/path/to/binary -a dosomething
```

Zatim, u prvom konzolu pritisnite "**s**", a zatim u zabeleženim akcijama naznačite da li želite da ignorišete, dozvolite ili nešto drugo. Kada završite, pritisnite "**f**" i novi profil će biti kreiran u _/etc/apparmor.d/path.to.binary_

{% hint style="info" %}
Koristeći strelice možete odabrati šta želite da dozvolite/odbijete/nešto drugo
{% endhint %}

### aa-easyprof

Takođe možete kreirati šablon apparmor profila za binarnu datoteku sa:

```bash
sudo aa-easyprof /path/to/binary
# vim:syntax=apparmor
# AppArmor policy for binary
# ###AUTHOR###
# ###COPYRIGHT###
# ###COMMENT###

#include <tunables/global>

# No template variables specified

"/path/to/binary" {
#include <abstractions/base>

# No abstractions specified

# No policy groups specified

# No read paths specified

# No write paths specified
}
```

{% hint style="info" %}
Napomena da prema zadanim postavkama u kreiranom profilu ništa nije dozvoljeno, tako da je sve odbijeno. Morate dodati linije poput `/etc/passwd r,` da biste omogućili čitanje binarnog `/etc/passwd` na primer.
{% endhint %}

Zatim možete **primeniti** novi profil sa

```bash
sudo apparmor_parser -a /etc/apparmor.d/path.to.binary
```

### Modifikovanje profila iz logova

Sledeći alat će čitati logove i pitati korisnika da li želi da dozvoli neke od detektovanih zabranjenih radnji:

```bash
sudo aa-logprof
```

{% hint style="info" %}
Koristeći strelice možete odabrati šta želite da dozvolite/odbijete/šta god
{% endhint %}

### Upravljanje profilom

```bash
#Main profile management commands
apparmor_parser -a /etc/apparmor.d/profile.name #Load a new profile in enforce mode
apparmor_parser -C /etc/apparmor.d/profile.name #Load a new profile in complain mode
apparmor_parser -r /etc/apparmor.d/profile.name #Replace existing profile
apparmor_parser -R /etc/apparmor.d/profile.name #Remove profile
```

## Dnevnici

Primer **AUDIT** i **DENIED** dnevnika iz _/var/log/audit/audit.log_ izvršnog fajla **`service_bin`**:

```bash
type=AVC msg=audit(1610061880.392:286): apparmor="AUDIT" operation="getattr" profile="/bin/rcat" name="/dev/pts/1" pid=954 comm="service_bin" requested_mask="r" fsuid=1000 ouid=1000
type=AVC msg=audit(1610061880.392:287): apparmor="DENIED" operation="open" profile="/bin/rcat" name="/etc/hosts" pid=954 comm="service_bin" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```

Možete dobiti ove informacije koristeći:

```bash
sudo aa-notify -s 1 -v
Profile: /bin/service_bin
Operation: open
Name: /etc/passwd
Denied: r
Logfile: /var/log/audit/audit.log

Profile: /bin/service_bin
Operation: open
Name: /etc/hosts
Denied: r
Logfile: /var/log/audit/audit.log

AppArmor denials: 2 (since Wed Jan  6 23:51:08 2021)
For more information, please see: https://wiki.ubuntu.com/DebuggingApparmor
```

## Apparmor u Dockeru

Primetite kako je podrazumevano učitan profil **docker-profile** za docker:

```bash
sudo aa-status
apparmor module is loaded.
50 profiles are loaded.
13 profiles are in enforce mode.
/sbin/dhclient
/usr/bin/lxc-start
/usr/lib/NetworkManager/nm-dhcp-client.action
/usr/lib/NetworkManager/nm-dhcp-helper
/usr/lib/chromium-browser/chromium-browser//browser_java
/usr/lib/chromium-browser/chromium-browser//browser_openjdk
/usr/lib/chromium-browser/chromium-browser//sanitized_helper
/usr/lib/connman/scripts/dhclient-script
docker-default
```

Podrazumevano, **Apparmor docker-default profil** se generiše sa [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

**docker-default profil sažetak**:

* **Pristup** svim **mrežama**
* **Nijedna sposobnost** nije definisana (Međutim, neke sposobnosti će doći iz uključivanja osnovnih osnovnih pravila, tj. #include \<abstractions/base>)
* **Pisanje** u bilo koji **/proc** fajl nije **dozvoljeno**
* Ostali **poddirektorijumi**/**fajlovi** od /**proc** i /**sys** su **odbijeni** pristup za čitanje/pisanje/zaključavanje/povezivanje/izvršavanje
* **Montiranje** nije dozvoljeno
* **Ptrace** se može pokrenuti samo na procesu koji je ograničen istim apparmor profilom

Jednom kada **pokrenete docker kontejner**, trebali biste videti sledeći izlaz:

```bash
1 processes are in enforce mode.
docker-default (825)
```

Imajte na umu da će **apparmor čak blokirati privilegije sposobnosti** koje su podrazumevano dodeljene kontejneru. Na primer, može **blokirati dozvolu za pisanje unutar /proc čak i ako je dodeljena SYS\_ADMIN sposobnost**, jer apparmor profil za docker podrazumevano odbija ovaj pristup:

```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined ubuntu /bin/bash
echo "" > /proc/stat
sh: 1: cannot create /proc/stat: Permission denied
```

Potrebno je **onemogućiti apparmor** kako biste zaobišli njegova ograničenja:

```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu /bin/bash
```

Napomena da će **AppArmor** po defaultu **onemogućiti kontejner da montira** foldere iznutra čak i sa SYS\_ADMIN mogućnostima.

Napomena da možete **dodati/ukloniti** **mogućnosti** kontejneru (ovo će i dalje biti ograničeno zaštitnim metodama kao što su **AppArmor** i **Seccomp**):

* `--cap-add=SYS_ADMIN` dodaje mogućnost `SYS_ADMIN`
* `--cap-add=ALL` dodaje sve mogućnosti
* `--cap-drop=ALL --cap-add=SYS_PTRACE` uklanja sve mogućnosti i samo dodaje `SYS_PTRACE`

{% hint style="info" %}
Obično, kada **otkrijete** da imate **privilegovanu mogućnost** dostupnu **unutar** Docker kontejnera **ali** neki deo **eksploita ne radi**, to je zato što docker **apparmor to sprečava**.
{% endhint %}

### Primer

(Primer sa [**ovde**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/))

Da bih ilustrovao funkcionalnost AppArmor-a, kreirao sam novi Docker profil "mydocker" sa dodatom sledećom linijom:

```
deny /etc/* w,   # deny write for all files directly in /etc (not in a subdir)
```

Da bismo aktivirali profil, potrebno je da uradimo sledeće:

```
sudo apparmor_parser -r -W mydocker
```

Da bismo prikazali profile, možemo koristiti sledeću komandu. Komanda ispod prikazuje moj novi AppArmor profil.

```
$ sudo apparmor_status  | grep mydocker
mydocker
```

Kao što je prikazano ispod, dobijamo grešku prilikom pokušaja promene "/etc/" jer AppArmor profil sprečava pristup za pisanje u "/etc".

```
$ docker run --rm -it --security-opt apparmor:mydocker -v ~/haproxy:/localhost busybox chmod 400 /etc/hostname
chmod: /etc/hostname: Permission denied
```

### AppArmor Docker Bypass1

Možete pronaći koji **apparmor profil pokreće kontejner** koristeći:

```bash
docker inspect 9d622d73a614 | grep lowpriv
"AppArmorProfile": "lowpriv",
"apparmor=lowpriv"
```

Zatim možete pokrenuti sledeću liniju da **pronađete tačan profil koji se koristi**:

```bash
find /etc/apparmor.d/ -name "*lowpriv*" -maxdepth 1 2>/dev/null
```

U čudnom slučaju možete **izmeniti apparmor docker profil i ponovo ga učitati**. Možete ukloniti ograničenja i "zaobići" ih.

### AppArmor Docker Bypass2

**AppArmor se zasniva na putanji**, što znači da čak i ako štiti fajlove unutar direktorijuma kao što je **`/proc`**, ako možete **konfigurisati kako će se kontejner pokrenuti**, možete **montirati** proc direktorijum domaćina unutar **`/host/proc`** i on više neće biti zaštićen od strane AppArmor-a.

### AppArmor Shebang Bypass

U [**ovom bagu**](https://bugs.launchpad.net/apparmor/+bug/1911431) možete videti primer kako **čak i ako sprečavate izvršavanje perla sa određenim resursima**, ako samo kreirate shell skriptu **navodeći** u prvom redu **`#!/usr/bin/perl`** i **izvršite fajl direktno**, moći ćete izvršiti šta god želite. Na primer:

```perl
echo '#!/usr/bin/perl
use POSIX qw(strftime);
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh"' > /tmp/test.pl
chmod +x /tmp/test.pl
/tmp/test.pl
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
