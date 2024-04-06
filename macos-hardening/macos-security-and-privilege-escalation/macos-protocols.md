# macOS Network Services & Protocols

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Usluge za daljinski pristup

Ovo su uobičajene macOS usluge za daljinski pristup.\
Možete omogućiti/onemogućiti ove usluge u `System Settings` --> `Sharing`

* **VNC**, poznat kao "Screen Sharing" (tcp:5900)
* **SSH**, nazvan "Remote Login" (tcp:22)
* **Apple Remote Desktop** (ARD), ili "Remote Management" (tcp:3283, tcp:5900)
* **AppleEvent**, poznat kao "Remote Apple Event" (tcp:3031)

Proverite da li je neka od ovih usluga omogućena pokretanjem:

```bash
rmMgmt=$(netstat -na | grep LISTEN | grep tcp46 | grep "*.3283" | wc -l);
scrShrng=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.5900" | wc -l);
flShrng=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | egrep "\*.88|\*.445|\*.548" | wc -l);
rLgn=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.22" | wc -l);
rAE=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.3031" | wc -l);
bmM=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.4488" | wc -l);
printf "\nThe following services are OFF if '0', or ON otherwise:\nScreen Sharing: %s\nFile Sharing: %s\nRemote Login: %s\nRemote Mgmt: %s\nRemote Apple Events: %s\nBack to My Mac: %s\n\n" "$scrShrng" "$flShrng" "$rLgn" "$rmMgmt" "$rAE" "$bmM";
```

### Pentestiranje ARD-a

Apple Remote Desktop (ARD) je unapređena verzija [Virtual Network Computing (VNC)](https://en.wikipedia.org/wiki/Virtual\_Network\_Computing) prilagođena za macOS, koja nudi dodatne funkcionalnosti. Značajna ranjivost u ARD-u je njegov metod autentifikacije za kontrolni ekran lozinke, koji koristi samo prvih 8 karaktera lozinke, što ga čini podložnim [brute force napadima](https://thudinh.blogspot.com/2017/09/brute-forcing-passwords-with-thc-hydra.html) pomoću alata kao što su Hydra ili [GoRedShell](https://github.com/ahhh/GoRedShell/), jer ne postoje podrazumevani limiti brzine.

Ranjive instance mogu se identifikovati korišćenjem **nmap**-ovog `vnc-info` skripta. Servisi koji podržavaju `VNC Authentication (2)` su posebno podložni brute force napadima zbog odsjecanja lozinke na 8 karaktera.

Da biste omogućili ARD za različite administrativne zadatke kao što su eskalacija privilegija, pristup grafičkom korisničkom interfejsu ili praćenje korisnika, koristite sledeću komandu:

```bash
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -activate -configure -allowAccessFor -allUsers -privs -all -clientopts -setmenuextra -menuextra yes
```

ARD pruža različite nivoe kontrole, uključujući posmatranje, deljenu kontrolu i potpunu kontrolu, sa sesijama koje traju čak i nakon promene korisničke lozinke. Omogućava slanje Unix komandi direktno, izvršavajući ih kao root za administrativne korisnike. Značajne funkcije su zakazivanje zadataka i pretraga udaljenog Spotlight-a, koje olakšavaju udaljene pretrage osetljivih datoteka na više mašina.

## Bonjour protokol

Bonjour, tehnologija dizajnirana od strane Apple-a, omogućava uređajima na istoj mreži da otkriju ponuđene usluge jedni drugih. Poznat i kao Rendezvous, Zero Configuration ili Zeroconf, omogućava uređaju da se pridruži TCP/IP mreži, automatski izabere IP adresu i emituje svoje usluge drugim mrežnim uređajima.

Zero Configuration Networking, koji pruža Bonjour, omogućava uređajima da:

* Automatski dobiju IP adresu čak i u odsustvu DHCP servera.
* Izvrše prevod imena u adresu bez potrebe za DNS serverom.
* Otkriju dostupne usluge na mreži.

Uređaji koji koriste Bonjour će sami dodeliti IP adresu iz opsega 169.254/16 i proveriti njenu jedinstvenost na mreži. Mac računari održavaju unos u rutiranju za ovu podmrežu, koji se može proveriti putem `netstat -rn | grep 169`.

Za DNS, Bonjour koristi Multicast DNS (mDNS) protokol. mDNS radi preko porta 5353/UDP, koristeći standardne DNS upite, ali ciljajući multicast adresu 224.0.0.251. Ovaj pristup omogućava da svi uređaji koji slušaju na mreži mogu da primaju i odgovaraju na upite, olakšavajući ažuriranje njihovih zapisa.

Prilikom pridruživanja mreži, svaki uređaj sam bira ime, koje obično završava sa .local, a može biti izvedeno iz imena hosta ili generisano nasumično.

Otkrivanje usluga unutar mreže olakšano je pomoću DNS Service Discovery (DNS-SD). Iskorišćavajući format DNS SRV zapisa, DNS-SD koristi DNS PTR zapise kako bi omogućio listanje više usluga. Klijent koji traži određenu uslugu će zatražiti PTR zapis za `<Usluga>.<Domen>`, a zauzvrat će dobiti listu PTR zapisa formatiranih kao `<Instanca>.<Usluga>.<Domen>` ako je usluga dostupna sa više hostova.

`dns-sd` alat može se koristiti za otkrivanje i oglašavanje mrežnih usluga. Evo nekoliko primera njegove upotrebe:

### Pretraga SSH usluga

Za pretragu SSH usluga na mreži koristi se sledeća komanda:

```bash
dns-sd -B _ssh._tcp
```

Ova komanda pokreće pretraživanje za \_ssh.\_tcp uslugama i prikazuje detalje kao što su vremenska oznaka, zastavice, interfejs, domen, tip usluge i ime instance.

### Oglašavanje HTTP usluge

Da biste oglašavali HTTP uslugu, možete koristiti:

```bash
dns-sd -R "Index" _http._tcp . 80 path=/index.html
```

Ova komanda registruje HTTP servis nazvan "Index" na portu 80 sa putanjom `/index.html`.

Zatim, da biste pretražili HTTP servise na mreži:

```bash
dns-sd -B _http._tcp
```

Kada se servis pokrene, on objavljuje svoju dostupnost svim uređajima u podmreži putem multicastiranja svoje prisutnosti. Uređaji zainteresovani za ove servise ne moraju slati zahteve, već jednostavno slušaju ove objave.

Za korisnički prijateljski interfejs, aplikacija **Discovery - DNS-SD Browser** dostupna na Apple App Store-u može vizualizovati servise koji se nude na lokalnoj mreži.

Alternativno, mogu se napisati prilagođeni skriptovi za pretraživanje i otkrivanje servisa koristeći biblioteku `python-zeroconf`. Skript [**python-zeroconf**](https://github.com/jstasiak/python-zeroconf) demonstrira kreiranje pretraživača servisa za `_http._tcp.local.` servise, ispisujući dodate ili uklonjene servise:

```python
from zeroconf import ServiceBrowser, Zeroconf

class MyListener:

def remove_service(self, zeroconf, type, name):
print("Service %s removed" % (name,))

def add_service(self, zeroconf, type, name):
info = zeroconf.get_service_info(type, name)
print("Service %s added, service info: %s" % (name, info))

zeroconf = Zeroconf()
listener = MyListener()
browser = ServiceBrowser(zeroconf, "_http._tcp.local.", listener)
try:
input("Press enter to exit...\n\n")
finally:
zeroconf.close()
```

### Onemogućavanje Bonjour-a

Ako postoje zabrinutosti u vezi sa sigurnošću ili drugi razlozi za onemogućavanje Bonjour-a, može se isključiti korišćenjem sledeće komande:

```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist
```

## Reference

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html**](https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
