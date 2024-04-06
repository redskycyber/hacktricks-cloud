# External Recon Methodology

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodiču PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ako vas zanima **hakerska karijera** i hakovanje onoga što se ne može hakovati - **mi zapošljavamo!** (_potrebno je tečno poznavanje poljskog jezika, kako pisano tako i govorno_).

{% embed url="https://www.stmcyber.com/careers" %}

## Otkrivanje resursa

> Rečeno vam je da sve što pripada nekoj kompaniji spada u opseg, i želite da saznate šta ta kompanija zapravo poseduje.

Cilj ove faze je da se dobiju sve **kompanije koje pripadaju glavnoj kompaniji** i zatim svi **resursi** ovih kompanija. Da bismo to postigli, uradićemo sledeće:

1. Pronaći akvizicije glavne kompanije, što će nam dati kompanije u opsegu.
2. Pronaći ASN (ako postoji) svake kompanije, što će nam dati IP opsege koje poseduje svaka kompanija.
3. Koristiti pretrage obrnutog whois-a da bismo tražili druge unose (nazive organizacija, domene...) povezane sa prvom (ovo se može raditi rekurzivno).
4. Koristiti druge tehnike poput shodan `org` i `ssl` filtera da bismo tražili druge resurse (trič za `ssl` se može raditi rekurzivno).

### **Akvizicije**

Prvo, treba da znamo koje **druge kompanije pripadaju glavnoj kompaniji**.\
Jedna opcija je posetiti [https://www.crunchbase.com/](https://www.crunchbase.com), **pretražiti** glavnu kompaniju, i **kliknuti** na "**akvizicije**". Tamo ćete videti druge kompanije koje je stekla glavna kompanija.\
Druga opcija je posetiti **Vikipedijinu** stranicu glavne kompanije i tražiti **akvizicije**.

> Ok, u ovom trenutku trebalo bi da znate sve kompanije u opsegu. Hajde da saznamo kako da pronađemo njihove resurse.

### **ASN-ovi**

Autonomni sistemski broj (**ASN**) je **jedinstveni broj** dodeljen **autonomnom sistemu** (AS) od strane **Internet Assigned Numbers Authority (IANA)**.\
AS se sastoji od **blokova** IP adresa koji imaju jasno definisanu politiku za pristupanje spoljnim mrežama i upravlja ih jedna organizacija, ali može biti sastavljen od više operatera.

Interesantno je saznati da li **kompanija ima dodeljen bilo koji ASN** kako bismo pronašli njene **IP opsege**. Bilo bi korisno izvršiti **test ranjivosti** protiv svih **hostova** unutar **opsega** i tražiti **domene** unutar ovih IP-ova.\
Možete **pretraživati** po imenu kompanije, po **IP**-u ili po **domenu** na [**https://bgp.he.net/**](https://bgp.he.net)**.**\
**Zavisno od regiona kompanije, ovi linkovi mogu biti korisni za prikupljanje više podataka:** [**AFRINIC**](https://www.afrinic.net) **(Afrika),** [**Arin**](https://www.arin.net/about/welcome/region/)**(Severna Amerika),** [**APNIC**](https://www.apnic.net) **(Azija),** [**LACNIC**](https://www.lacnic.net) **(Latinska Amerika),** [**RIPE NCC**](https://www.ripe.net) **(Evropa). U svakom slučaju, verovatno su svi** korisni podaci **(IP opsezi i Whois)** već dostupni na prvom linku.

```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```

Takođe, [**BBOT**](https://github.com/blacklanternsecurity/bbot)**-ova** enumeracija poddomena automatski agregira i sumira ASN-ove na kraju skeniranja.

```bash
bbot -t tesla.com -f subdomain-enum
...
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.244.131.0/24      | 5            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS16509  | 54.148.0.0/15       | 4            | AMAZON-02      | Amazon.com, Inc.           | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.45.124.0/24       | 3            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.32.0.0/12         | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.0.0.0/9           | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+

```

Možete pronaći IP opsege organizacije takođe koristeći [http://asnlookup.com/](http://asnlookup.com) (ima besplatan API).\
Možete pronaći IP i ASN domena koristeći [http://ipv4info.com/](http://ipv4info.com).

### **Traženje ranjivosti**

U ovom trenutku znamo **sve resurse unutar opsega**, pa ako vam je dozvoljeno, možete pokrenuti neki **skener ranjivosti** (Nessus, OpenVAS) na svim hostovima.\
Takođe, možete pokrenuti neke [**skeniranje portova**](../pentesting-network/#discovering-hosts-from-the-outside) **ili koristiti usluge kao što je** shodan **da biste pronašli** otvorene portove **i, u zavisnosti od onoga što pronađete, trebalo bi da** pogledate u ovoj knjizi kako da pentestirate nekoliko mogućih servisa koji se izvršavaju.\
**Takođe, vredi pomenuti da možete pripremiti neke** liste podrazumevanih korisničkih imena **i** lozinki **i pokušati da** probijete servise sa [https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray).

## Domeni

> Znamo sve kompanije unutar opsega i njihove resurse, vreme je da pronađemo domene unutar opsega.

_Molimo, imajte na umu da u sledećim predloženim tehnikama takođe možete pronaći poddomene i te informacije ne bi trebalo potcenjivati._

Prvo biste trebali potražiti **glavni domen(e)** svake kompanije. Na primer, za _Tesla Inc._ biće _tesla.com_.

### **Obrnuti DNS**

Kada ste pronašli sve IP opsege domena, možete pokušati da izvršite **obrnute DNS upite** na tim **IP-ovima kako biste pronašli više domena unutar opsega**. Pokušajte da koristite neki DNS server žrtve ili neki dobro poznati DNS server (1.1.1.1, 8.8.8.8)

```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```

### **Obrnuti Whois (petlja)**

Unutar **whois** informacija možete pronaći mnogo zanimljivih **podataka** poput **imenovanja organizacije**, **adrese**, **emailova**, brojeva telefona... Ali ono što je još interesantnije je da možete pronaći **više resursa povezanih sa kompanijom** ako izvršite **obrnute whois pretrage po bilo kojem od tih polja** (na primer, druge whois registre gde se isti email pojavljuje).\
Možete koristiti online alate poput:

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **Besplatno**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **Besplatno**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **Besplatno**
* [https://www.whoxy.com/](https://www.whoxy.com) - **Besplatno** web, nije besplatan API.
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - Nije besplatno
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - Nije besplatno (samo **100 besplatnih** pretraga)
* [https://www.domainiq.com/](https://www.domainiq.com) - Nije besplatno

Možete automatizovati ovaj zadatak koristeći [**DomLink** ](https://github.com/vysecurity/DomLink)(zahteva whoxy API ključ).\
Takođe možete izvršiti automatsko otkrivanje obrnutog whois-a sa [amass](https://github.com/OWASP/Amass): `amass intel -d tesla.com -whois`

**Imajte na umu da možete koristiti ovu tehniku da otkrijete više imena domena svaki put kada pronađete novi domen.**

### **Trackeri**

Ako pronađete **isti ID istog trackera** na 2 različite stranice, možete pretpostaviti da **obe stranice** upravlja **isti tim**.\
Na primer, ako vidite isti **Google Analytics ID** ili isti **Adsense ID** na nekoliko stranica.

Postoje neke stranice i alati koji vam omogućavaju pretragu po ovim trackerima i više:

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **Favicon**

Da li ste znali da možemo pronaći povezane domene i pod-domenove našeg cilja tražeći isti hash ikone favicona? To je upravo ono što alat [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py) napravljen od strane [@m4ll0k2](https://twitter.com/m4ll0k2) radi. Evo kako ga koristiti:

```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```

![favihash - otkrijte domene sa istim hešom ikone favicon-a](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

Jednostavno rečeno, favihash će nam omogućiti da otkrijemo domene koje imaju isti heš ikone favicon-a kao naš cilj.

Osim toga, možete takođe pretraživati tehnologije koristeći heš favicon-a kako je objašnjeno u [**ovom blog postu**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139). To znači da ako znate **heš favicon-a ranjive verzije web tehnologije** možete pretražiti u shodanu i **pronaći više ranjivih mesta**:

```bash
shodan search org:"Target" http.favicon.hash:116323821 --fields ip_str,port --separator " " | awk '{print $1":"$2}'
```

Evo kako možete **izračunati heš favicon-a** veb stranice:

```python
import mmh3
import requests
import codecs

def fav_hash(url):
response = requests.get(url)
favicon = codecs.encode(response.content,"base64")
fhash = mmh3.hash(favicon)
print(f"{url} : {fhash}")
return fhash
```

### **Autorsko pravo / Jedinstveni string**

Pretražite unutar web stranica **stringove koji bi mogli biti deljeni između različitih veb sajtova u istoj organizaciji**. **String autorskog prava** može biti dobar primer. Zatim pretražite taj string na **google-u**, u drugim **pregledačima** ili čak na **shodan-u**: `shodan search http.html:"String autorskog prava"`

### **CRT vreme**

Često je uobičajeno imati cron posao kao što je

```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```

### **Spoljni rekon metodologija**

Da biste obnovili sve sertifikate domena na serveru. To znači da čak i ako CA koji se koristi za ovo ne postavlja vreme kada je generisan u Vremenu važnosti, moguće je **pronaći domene koje pripadaju istoj kompaniji u logovima transparentnosti sertifikata**.\
Pogledajte ovaj [**članak za više informacija**](https://swarm.ptsecurity.com/discovering-domains-via-a-time-correlation-attack/).

### **Pasivno preuzimanje**

Očigledno je da je uobičajeno da ljudi dodeljuju poddomene IP adresama koje pripadaju provajderima oblaka i u nekom trenutku **izgube tu IP adresu ali zaborave da uklone DNS zapis**. Stoga, samo **pokretanje virtuelne mašine** u oblaku (kao što je Digital Ocean) zapravo će **preuzeti neke poddomene**.

[**Ovaj post**](https://kmsec.uk/blog/passive-takeover/) objašnjava priču o tome i predlaže skriptu koja **pokreće virtuelnu mašinu u DigitalOcean-u**, **dobija** IPv4 **nove mašine i traži u Virustotal-u zapise poddomena** koji na nju pokazuju.

### **Drugi načini**

**Imajte na umu da možete koristiti ovu tehniku da otkrijete više imena domena svaki put kada pronađete novi domen.**

**Shodan**

Kako već znate ime organizacije koja poseduje IP prostor. Možete pretražiti te podatke u shodan-u koristeći: `org:"Tesla, Inc."` Proverite pronađene hostove za nove neočekivane domene u TLS sertifikatu.

Mogli biste pristupiti **TLS sertifikatu** glavne web stranice, dobiti **ime organizacije** i zatim tražiti to ime unutar **TLS sertifikata** svih web stranica poznatih od strane **shodan-a** sa filterom: `ssl:"Tesla Motors"` ili koristiti alat poput [**sslsearch**](https://github.com/HarshVaragiya/sslsearch).

**Assetfinder**

[**Assetfinder**](https://github.com/tomnomnom/assetfinder) je alat koji traži **domene povezane** sa glavnim domenom i **poddomene** od njih, prilično neverovatan.

### **Traženje ranjivosti**

Proverite da li postoji [preuzimanje domena](../../pentesting-web/domain-subdomain-takeover.md#domain-takeover). Možda neka kompanija **koristi neki domen** ali su **izgubili vlasništvo**. Samo ga registrujte (ako je dovoljno jeftino) i obavestite kompaniju.

Ako pronađete bilo koji **domen sa drugačijom IP adresom** od onih koje ste već pronašli u otkrivanju resursa, trebalo bi da izvršite **osnovno skeniranje ranjivosti** (koristeći Nessus ili OpenVAS) i neko [**skeniranje portova**](../pentesting-network/#discovering-hosts-from-the-outside) sa **nmap/masscan/shodan**. Zavisno od toga koje usluge se izvršavaju, možete pronaći u **ovoj knjizi neke trikove za "napad" na njih**.\
_Napomena da se ponekad domen nalazi unutar IP adrese koja nije pod kontrolom klijenta, pa nije u opsegu, budite oprezni._

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**Savet za bug bounty**: **Prijavite se** za **Intigriti**, premijum **platformu za bug bounty kreiranu od hakera, za hakere**! Pridružite nam se na [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) danas, i počnite da zarađujete nagrade do **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

## Poddomeni

> Znamo sve kompanije unutar opsega, sve resurse svake kompanije i sve domene povezane sa kompanijama.

Vreme je da pronađemo sve moguće poddomene svakog pronađenog domena.

### **DNS**

Pokušajmo da dobijemo **poddomene** iz **DNS** zapisa. Takođe bismo trebali pokušati za **Zone Transfer** (Ako je ranjiv, trebalo bi da prijavite).

```bash
dnsrecon -a -d tesla.com
```

### **OSINT**

Najbrži način da dobijete mnogo poddomena je pretraga u eksternim izvorima. Najkorišćeniji **alati** su sledeći (za bolje rezultate konfigurišite API ključeve):

* [**BBOT**](https://github.com/blacklanternsecurity/bbot)

```bash
# subdomains
bbot -t tesla.com -f subdomain-enum

# subdomains (passive only)
bbot -t tesla.com -f subdomain-enum -rf passive

# subdomains + port scan + web screenshots
bbot -t tesla.com -f subdomain-enum -m naabu gowitness -n my_scan -o .
```

* [**Amass**](https://github.com/OWASP/Amass)

```bash
amass enum [-active] [-ip] -d tesla.com
amass enum -d tesla.com | grep tesla.com # To just list subdomains
```

* [**subfinder**](https://github.com/projectdiscovery/subfinder)

```bash
# Subfinder, use -silent to only have subdomains in the output
./subfinder-linux-amd64 -d tesla.com [-silent]
```

* [**findomain**](https://github.com/Edu4rdSHL/findomain/)

```bash
# findomain, use -silent to only have subdomains in the output
./findomain-linux -t tesla.com [--quiet]
```

* [**OneForAll**](https://github.com/shmilylty/OneForAll/tree/master/docs/sr)

```bash
python3 oneforall.py --target tesla.com [--dns False] [--req False] [--brute False] run
```

* [**assetfinder**](https://github.com/tomnomnom/assetfinder)

```bash
assetfinder --subs-only <domain>
```

* [**Sudomy**](https://github.com/Screetsec/Sudomy)

```bash
# It requires that you create a sudomy.api file with API keys
sudomy -d tesla.com
```

* [**vita**](https://github.com/junnlikestea/vita)

```
vita -d tesla.com
```

* [**theHarvester**](https://github.com/laramies/theHarvester)

```bash
theHarvester -d tesla.com -b "anubis, baidu, bing, binaryedge, bingapi, bufferoverun, censys, certspotter, crtsh, dnsdumpster, duckduckgo, fullhunt, github-code, google, hackertarget, hunter, intelx, linkedin, linkedin_links, n45ht, omnisint, otx, pentesttools, projectdiscovery, qwant, rapiddns, rocketreach, securityTrails, spyse, sublist3r, threatcrowd, threatminer, trello, twitter, urlscan, virustotal, yahoo, zoomeye"
```

Postoje **drugi zanimljivi alati/API-ji** koji, iako nisu direktno specijalizovani za pronalaženje poddomena, mogu biti korisni za pronalaženje poddomena, poput:

* [**Crobat**](https://github.com/cgboal/sonarsearch)**:** Koristi API [https://sonar.omnisint.io](https://sonar.omnisint.io) za dobijanje poddomena

```bash
# Get list of subdomains in output from the API
## This is the API the crobat tool will use
curl https://sonar.omnisint.io/subdomains/tesla.com | jq -r ".[]"
```

* [**JLDC besplatni API**](https://jldc.me/anubis/subdomains/google.com)

```bash
curl https://jldc.me/anubis/subdomains/tesla.com | jq -r ".[]"
```

* [**RapidDNS**](https://rapiddns.io) besplatni API

```bash
# Get Domains from rapiddns free API
rapiddns(){
curl -s "https://rapiddns.io/subdomain/$1?full=1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
rapiddns tesla.com
```

* [**https://crt.sh/**](https://crt.sh)

```bash
# Get Domains from crt free API
crt(){
curl -s "https://crt.sh/?q=%25.$1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
crt tesla.com
```

* [**gau**](https://github.com/lc/gau)**:** preuzima poznate URL-ove sa AlienVault-ove Open Threat Exchange, Wayback Machine-a i Common Crawl-a za bilo koji dati domen.

```bash
# Get subdomains from GAUs found URLs
gau --subs tesla.com | cut -d "/" -f 3 | sort -u
```

* [**SubDomainizer**](https://github.com/nsonaniya2010/SubDomainizer) **&** [**subscraper**](https://github.com/Cillian-Collins/subscraper): Oni pretražuju web u potrazi za JS fajlovima i iz njih izvlače poddomene.

```bash
# Get only subdomains from SubDomainizer
python3 SubDomainizer.py -u https://tesla.com | grep tesla.com

# Get only subdomains from subscraper, this already perform recursion over the found results
python subscraper.py -u tesla.com | grep tesla.com | cut -d " " -f
```

* [**Shodan**](https://www.shodan.io/)

```bash
# Get info about the domain
shodan domain <domain>
# Get other pages with links to subdomains
shodan search "http.html:help.domain.com"
```

* [**Censys alat za pronalaženje poddomena**](https://github.com/christophetd/censys-subdomain-finder)

```bash
export CENSYS_API_ID=...
export CENSYS_API_SECRET=...
python3 censys-subdomain-finder.py tesla.com
```

* [**DomainTrail.py**](https://github.com/gatete/DomainTrail)

```bash
python3 DomainTrail.py -d example.com
```

* [**securitytrails.com**](https://securitytrails.com/) ima besplatnu API za pretragu poddomena i istoriju IP adresa
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

Ovaj projekat nudi **besplatno sve poddomene vezane za programe bug-bounty-a**. Možete pristupiti ovim podacima i koristeći [chaospy](https://github.com/dr-0x0x/chaospy) ili pristupiti opsegu korišćenom od strane ovog projekta [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

Možete pronaći **poređenje** mnogih ovih alata ovde: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **DNS Brute force**

Pokušajmo da pronađemo nove **poddomene** brute-forcing DNS servere koristeći moguća imena poddomena.

Za ovu akciju će vam biti potrebne neke **uobičajene liste reči za poddomene kao što su**:

* [https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
* [https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt)
* [https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip](https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip)
* [https://github.com/pentester-io/commonspeak](https://github.com/pentester-io/commonspeak)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

I takođe IP adrese dobrih DNS resolvera. Da biste generisali listu pouzdanih DNS resolvera, možete preuzeti resolvere sa [https://public-dns.info/nameservers-all.txt](https://public-dns.info/nameservers-all.txt) i koristiti [**dnsvalidator**](https://github.com/vortexau/dnsvalidator) da ih filtrirate. Ili možete koristiti: [https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

Najpreporučeniji alati za DNS brute-force su:

* [**massdns**](https://github.com/blechschmidt/massdns): Ovo je bio prvi alat koji je izveo efikasan DNS brute-force. Veoma je brz, međutim sklon je lažnim pozitivima.

```bash
sed 's/$/.domain.com/' subdomains.txt > bf-subdomains.txt
./massdns -r resolvers.txt -w /tmp/results.txt bf-subdomains.txt
grep -E "tesla.com. [0-9]+ IN A .+" /tmp/results.txt
```

* [**gobuster**](https://github.com/OJ/gobuster): Mislim da ovaj koristi samo 1 rešavač

```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```

* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) je omotač oko `massdns`, napisan u go-u, koji vam omogućava da nabrojite validne poddomene korišćenjem aktivnog brute force-a, kao i da rešite poddomene sa rukovanjem wildcard-ima i jednostavnom podrškom za unos-izlaz.

```
shuffledns -d example.com -list example-subdomains.txt -r resolvers.txt
```

* [**puredns**](https://github.com/d3mondev/puredns): Takođe koristi `massdns`.

```
puredns bruteforce all.txt domain.com
```

* [**aiodnsbrute**](https://github.com/blark/aiodnsbrute) koristi asyncio za asinhrono grubo pretraživanje imena domena.

```
aiodnsbrute -r resolvers -w wordlist.txt -vv -t 1024 domain.com
```

### Druga runda Brute-Force napada na DNS

Nakon što ste pronašli poddomene korišćenjem otvorenih izvora i brute-force tehnike, možete generisati varijacije pronađenih poddomena kako biste pokušali pronaći još više. Za ovu svrhu korisni su neki alati:

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** Generiše permutacije domena i poddomena.

```bash
cat subdomains.txt | dnsgen -
```

* [**goaltdns**](https://github.com/subfinder/goaltdns): Dati domeni i poddomeni generišu permutacije.
* Možete dobiti goaltdns permutacije **wordlist** ovde: [**here**](https://github.com/subfinder/goaltdns/blob/master/words.txt).

```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```

* [**gotator**](https://github.com/Josue87/gotator)**:** Dati domeni i poddomeni generišu permutacije. Ako nije naznačena datoteka permutacija, gotator će koristiti svoju.

```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```

* [**altdns**](https://github.com/infosec-au/altdns): Pored generisanja permutacija poddomena, može takođe pokušati da ih reši (ali je bolje koristiti prethodno komentarisane alate).
* Možete preuzeti altdns permutacije **wordlist** [**ovde**](https://github.com/infosec-au/altdns/blob/master/words.txt).

```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```

* [**dmut**](https://github.com/bp0lr/dmut): Još jedan alat za izvođenje permutacija, mutacija i izmena poddomena. Ovaj alat će grubo forsirati rezultat (ne podržava dns wild card).
* Možete preuzeti dmut permutacije liste reči [**ovde**](https://raw.githubusercontent.com/bp0lr/dmut/main/words.txt).

```bash
cat subdomains.txt | dmut -d /tmp/words-permutations.txt -w 100 \
--dns-errorLimit 10 --use-pb --verbose -s /tmp/resolvers-trusted.txt
```

* [**alterx**](https://github.com/projectdiscovery/alterx)**:** Zasnovan na domenu, **generiše nove potencijalne poddomene** na osnovu naznačenih obrazaca kako bi pokušao da otkrije više poddomena.

#### Pametna generacija permutacija

* [**regulator**](https://github.com/cramppet/regulator): Za više informacija pročitajte ovaj [**post**](https://cramppet.github.io/regulator/index.html) ali će u osnovi uzeti **glavne delove** otkrivenih poddomena i mešati ih kako bi pronašao više poddomena.

```bash
python3 main.py adobe.com adobe adobe.rules
make_brute_list.sh adobe.rules adobe.brute
puredns resolve adobe.brute --write adobe.valid
```

* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_ je fuzzer za grubu silu poddomena uparen sa izuzetno jednostalim, ali efikasnim DNS vođenim algoritmom. Koristi pruženi set ulaznih podataka, poput prilagođene liste reči ili istorijskih DNS/TLS zapisa, kako bi tačno sintetisao više odgovarajućih imena domena i dalje ih proširio u petlji na osnovu prikupljenih informacija tokom skeniranja DNS-a.

```
echo www | subzuf facebook.com
```

### **Radni tok otkrivanja poddomena**

Proverite ovaj blog post koji sam napisao o tome kako **automatizovati otkrivanje poddomena** sa domena koristeći **Trickest radne tokove** tako da ne moram ručno pokretati gomilu alata na svom računaru:

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### **VHosts / Virtuelni hostovi**

Ako pronađete IP adresu koja sadrži **jednu ili više veb stranica** koje pripadaju poddomenima, možete pokušati **pronaći druge poddomene sa veb stranicama na toj IP adresi** tražeći u **OSINT izvorima** domene na IP adresi ili **brute-force metodom VHost imena domena na toj IP adresi**.

#### OSINT

Možete pronaći neke **VHostove na IP adresama koristeći** [**HostHunter**](https://github.com/SpiderLabs/HostHunter) **ili druge API-je**.

**Brute Force**

Ako sumnjate da se neka poddomena može sakriti na veb serveru, možete pokušati da je brute force-ujete:

```bash
ffuf -c -w /path/to/wordlist -u http://victim.com -H "Host: FUZZ.victim.com"

gobuster vhost -u https://mysite.com -t 50 -w subdomains.txt

wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --hc 400,404,403 -H "Host: FUZZ.example.com" -u http://example.com -t 100

#From https://github.com/allyshka/vhostbrute
vhostbrute.py --url="example.com" --remoteip="10.1.1.15" --base="www.example.com" --vhosts="vhosts_full.list"

#https://github.com/codingo/VHostScan
VHostScan -t example.com
```

{% hint style="info" %}
Ovom tehnikom možda čak možete pristupiti internim/skrivenim endpointima.
{% endhint %}

### **CORS Brute Force**

Ponekad ćete pronaći stranice koje vraćaju samo zaglavlje _**Access-Control-Allow-Origin**_ kada je validna domena/poddomena postavljena u zaglavlju _**Origin**_. U ovim scenarijima, možete zloupotrebiti ovu ponašanje da **otkrijete** nove **poddomene**.

```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```

### **Brute Force za Buckets**

Prilikom traženja **poddomena**, obratite pažnju da li je usmeren ka nekoj vrsti **bucket**-a, i u tom slučaju [**proverite dozvole**](../../network-services-pentesting/pentesting-web/buckets/)**.**\
Takođe, s obzirom da ćete u ovom trenutku znati sve domene unutar opsega, pokušajte [**brute force mogućih imena bucket-a i proverite dozvole**](../../network-services-pentesting/pentesting-web/buckets/).

### **Monitorisanje**

Možete **pratiti** da li su **novi poddomeni** domena kreirani praćenjem **Certificate Transparency** logova koje radi [**sublert**](https://github.com/yassineaboukir/sublert/blob/master/sublert.py).

### **Traženje ranjivosti**

Proverite moguće [**preuzimanje poddomena**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover).\
Ako **poddomen** usmerava ka nekom **S3 bucket**-u, [**proverite dozvole**](../../network-services-pentesting/pentesting-web/buckets/).

Ako pronađete bilo koji **poddomen sa IP adresom različitom** od onih koje ste već pronašli u otkrivanju resursa, trebalo bi da izvršite **osnovno skeniranje ranjivosti** (korišćenjem Nessus-a ili OpenVAS-a) i neko [**skeniranje portova**](../pentesting-network/#discovering-hosts-from-the-outside) sa **nmap/masscan/shodan**. Zavisno od toga koje usluge se izvršavaju, možete pronaći u **ovoj knjizi neke trikove za "napad" na njih**.\
_Napomena da se ponekad poddomen nalazi na IP adresi koja nije pod kontrolom klijenta, pa nije u opsegu, budite oprezni._

## IP adrese

U početnim koracima možda ste **pronašli neke opsege IP adresa, domene i poddomene**.\
Vreme je da **prikupite sve IP adrese iz tih opsega** i za **domene/poddomene (DNS upiti).**

Korišćenjem usluga sledećih **besplatnih API-ja** takođe možete pronaći **prethodne IP adrese koje su korišćene od strane domena i poddomena**. Te IP adrese možda i dalje pripadaju klijentu (i možda vam omoguće da pronađete [**CloudFlare zaobilaze**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md))

* [**https://securitytrails.com/**](https://securitytrails.com/)

Takođe možete proveriti domene koje usmeravaju ka određenoj IP adresi korišćenjem alata [**hakip2host**](https://github.com/hakluke/hakip2host)

### **Traženje ranjivosti**

**Skenirajte portove svih IP adresa koje ne pripadaju CDN-ovima** (jer verovatno nećete pronaći ništa zanimljivo tamo). U otkrivenim pokrenutim uslugama možda ćete biti **u mogućnosti da pronađete ranjivosti**.

**Pronađite** [**vodič**](../pentesting-network/) **o tome kako skenirati hostove.**

## Lov na web servere

> Pronašli smo sve kompanije i njihove resurse i znamo opsege IP adresa, domene i poddomene unutar opsega. Vreme je da tražimo web servere.

U prethodnim koracima verovatno ste već izvršili neko **rekonnoitering IP adresa i otkrivenih domena**, tako da možda već imate **pronašli sve moguće web servere**. Međutim, ako niste, sada ćemo videti neke **brze trikove za traženje web servera** unutar opsega.

Molimo, imajte na umu da će ovo biti **usmereno na otkrivanje web aplikacija**, pa biste trebali takođe **izvršiti skeniranje ranjivosti** i **portova** takođe (**ako je dozvoljeno** u opsegu).

**Brz način** za otkrivanje **otvorenih portova** koji se odnose na **web** servere korišćenjem [**masscan** možete pronaći ovde](../pentesting-network/#http-port-discovery).\
Još jedan koristan alat za traženje web servera je [**httprobe**](https://github.com/tomnomnom/httprobe)**,** [**fprobe**](https://github.com/theblackturtle/fprobe) i [**httpx**](https://github.com/projectdiscovery/httpx). Jednostavno prosledite listu domena i pokušaće da se poveže na port 80 (http) i 443 (https). Dodatno, možete naznačiti da pokuša i druge portove:

```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```

### **Screenshots**

Sada kada ste otkrili **sve veb servere** prisutne u opsegu (među **IP adresama** kompanije i svim **domenima** i **poddomenima**), verovatno **ne znate odakle da počnete**. Dakle, hajde da to učinimo jednostavnim i počnemo tako što ćemo napraviti snimke ekrana svih njih. Samo **pogledom** na **glavnu stranicu** možete pronaći **čudne** krajnje tačke koje su više **sklone** da budu **ranjive**.

Da biste sproveli predloženu ideju, možete koristiti [**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness), [**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot), [**Aquatone**](https://github.com/michenriksen/aquatone), [**Shutter**](https://shutter-project.org/downloads/third-party-packages/) ili [**webscreenshot**](https://github.com/maaaaz/webscreenshot)**.**

Osim toga, možete koristiti [**eyeballer**](https://github.com/BishopFox/eyeballer) da pregleda sve **snimke ekrana** i kaže vam **šta verovatno sadrži ranjivosti**, a šta ne.

## Javna Cloud sredstva

Da biste pronašli potencijalna cloud sredstva koja pripadaju kompaniji, trebalo bi da **počnete sa listom ključnih reči koje identifikuju tu kompaniju**. Na primer, za kripto kompaniju možete koristiti reči poput: `"kripto", "novčanik", "dao", "<ime_domena>", <"imeni_poddomena">`.

Takođe će vam biti potrebne liste reči koje se često koriste u **spremnicima**:

* [https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt](https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt)
* [https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt](https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt)
* [https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt](https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt)

Zatim, sa tim rečima trebalo bi da generišete **permutacije** (proverite [**Drugo kolo DNS Brute-Force**](./#second-dns-bruteforce-round) za više informacija).

Sa rezultirajućim listama reči možete koristiti alate poput [**cloud\_enum**](https://github.com/initstring/cloud\_enum)**,** [**CloudScraper**](https://github.com/jordanpotti/CloudScraper)**,** [**cloudlist**](https://github.com/projectdiscovery/cloudlist) **ili** [**S3Scanner**](https://github.com/sa7mon/S3Scanner)**.**

Zapamtite da prilikom traženja Cloud sredstava treba **tražiti više od samo spremnika u AWS**.

### **Traženje ranjivosti**

Ako pronađete stvari poput **otvorenih spremnika ili izloženih cloud funkcija**, trebalo bi da im **pristupite** i pokušate da vidite šta vam nude i da li ih možete zloupotrebiti.

## Emailovi

Sa **domenima** i **poddomenima** unutar opsega, praktično imate sve što vam je **potrebno da počnete tražiti emailove**. Ovo su **API-ji** i **alati** koji su najbolje funkcionisali za mene u pronalaženju emailova kompanije:

* [**theHarvester**](https://github.com/laramies/theHarvester) - sa API-ima
* API od [**https://hunter.io/**](https://hunter.io/) (besplatna verzija)
* API od [**https://app.snov.io/**](https://app.snov.io/) (besplatna verzija)
* API od [**https://minelead.io/**](https://minelead.io/) (besplatna verzija)

### **Traženje ranjivosti**

Emailovi će kasnije biti korisni za **bruteforce web prijava i autentikacione servise** (kao što je SSH). Takođe su potrebni za **fishing**. Osim toga, ovi API-ji će vam pružiti još više **informacija o osobi** iza emaila, što je korisno za kampanju phishinga.

## Curenje podataka o akreditacijama

Sa **domenima**, **poddomenima** i **emailovima** možete početi tražiti procurele akreditacije iz prošlosti koje pripadaju tim emailovima:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https://www.dehashed.com/)

### **Traženje ranjivosti**

Ako pronađete **validne procurele** akreditacije, to je veoma laka pobeda.

## Curenje tajni

Curenje akreditacija je povezano sa hakovanjem kompanija gde je **osetljive informacije procurene i prodate**. Međutim, kompanije mogu biti pogođene i **drugim curenjima** čije informacije nisu u tim bazama podataka:

### Github Curenja

Akreditacije i API-ji mogu biti procureni u **javnom repozitorijumu** **kompanije** ili **korisnika** koji rade za tu github kompaniju.\
Možete koristiti **alat** [**Leakos**](https://github.com/carlospolop/Leakos) da **preuzmete** sve **javne repozitorijume** organizacije i njenih **developer-a** i automatski pokrenete [**gitleaks**](https://github.com/zricethezav/gitleaks) nad njima.

**Leakos** takođe može se koristiti za pokretanje **gitleaks** nad svim **tekstualnim** URL-ovima koje mu prosledite jer ponekad **veb stranice takođe sadrže tajne**.

#### Github Dorks

Proverite takođe ovu **stranicu** za potencijalne **github dorks** koje takođe možete tražiti u organizaciji koju napadate:

{% content-ref url="github-leaked-secrets.md" %}
[github-leaked-secrets.md](github-leaked-secrets.md)
{% endcontent-ref %}

### Paste Curenja

Ponekad napadači ili samo radnici će **objaviti sadržaj kompanije na sajtu za paste**. To može ili ne mora sadržati **osetljive informacije**, ali je veoma interesantno tražiti ih.\
Možete koristiti alat [**Pastos**](https://github.com/carlospolop/Pastos) da pretražujete više od 80 sajtova za paste istovremeno.

### Google Dorks

Stari, ali zlatni google dorks uvek su korisni za pronalaženje **izloženih informacija koje ne bi trebalo da budu tamo**. Jedini problem je što [**google-hacking-database**](https://www.exploit-db.com/google-hacking-database) sadrži nekoliko **hiljada** mogućih upita koje ne možete pokrenuti ručno. Dakle, možete odabrati svojih 10 omiljenih ili možete koristiti **alat poput** [**Gorks**](https://github.com/carlospolop/Gorks) **da ih pokrenete sve**.

_Napomena da alati koji očekuju da pokrenu celu bazu podataka koristeći obični Google pretraživač nikada neće završiti jer će vas Google vrlo brzo blokirati._

### **Traženje ranjivosti**

Ako pronađete **validne procurele** akreditacije ili API tokena, to je veoma laka pobeda.

## Javne ranjivosti koda

Ako ste otkrili da kompanija ima **otvoren kod**, možete ga **analizirati** i tražiti **ranjivosti** u njemu.

**Zavisno o jeziku**, postoje različiti **alati** koje možete koristiti:

{% content-ref url="../../network-services-pentesting/pentesting-web/code-review-tools.md" %}
[code-review-tools.md](../../network-services-pentesting/pentesting-web/code-review-tools.md)
{% endcontent-ref %}

Postoje i besplatne usluge koje vam omogućavaju da **skenirate javne repozitorijume**, kao što su:

* [**Snyk**](https://app.snyk.io/)

## [**Metodologija testiranja veb aplikacija**](../../network-services-pentesting/pentesting-web/)

**Većina ranjivosti** otkrivenih od strane lovaca na bagove nalazi se unutar **veb aplikacija**, pa bih u ovom trenutku želeo da govorim o **metodologiji testiranja veb aplikacija**, a možete [**pronaći ove informacije ovde**](../../network-services-pentesting/pentesting-web/).

Takođe želim da posebno pomenem sekciju [**Alati otvorenog koda za automatsko skeniranje veb aplikacija**](../../network-services-pentesting/pentesting-web/#automatic-scanners), jer, iako ne treba očekivati da će pronaći veoma osetljive ranjivosti, korisni su za implementaciju u **tokove rada kako bi se dobile neke početne informacije o vebu.**

## Rekapitulacija

> Čestitam! Do ovog trenutka već ste obavili **svu osnovnu enumeraciju**. Da, osnovnu, jer se može obaviti mnogo više enumeracije (videćemo više trikova kasnije).

Dakle, već ste:

1. Pronašli sve **kompanije** unutar opsega
2. Pronašli sve **resurse** koji pripadaju kompanijama (i obavili skeniranje ranjivosti ako je u opsegu)
3. Pronašli sve **domene** koje pripadaju kompanijama
4. Pronašli sve **poddomene** domena (bilo preuzimanja poddomena?)
5. Pronašli sve **IP adrese** (iz i **ne iz CDN-a**) unutar opsega.
6. Pronašli sve **veb servere** i napravili **screenshot** (ima li nešto čudno vredno dubljeg pregleda?)
7. Pronašli sve **potencijalne javne cloud resurse** koji pripadaju kompaniji.
8. **Emailove**, **procurele akreditive** i **procurele tajne** koje bi vam mogle doneti **veliku pobedu veoma lako**.
9. **Testirali sve vebove koje ste pronašli**

## **Alati za potpunu automatsku rekonstrukciju**

Postoji nekoliko alata koji će obaviti deo predloženih akcija protiv datog opsega.

* [**https://github.com/yogeshojha/rengine**](https://github.com/yogeshojha/rengine)
* [**https://github.com/j3ssie/Osmedeus**](https://github.com/j3ssie/Osmedeus)
* [**https://github.com/six2dez/reconftw**](https://github.com/six2dez/reconftw)
* [**https://github.com/hackerspider1/EchoPwn**](https://github.com/hackerspider1/EchoPwn) - Malo zastareo i nije ažuriran

## **Reference**

* Svi besplatni kursevi od [**@Jhaddix**](https://twitter.com/Jhaddix) poput [**Metodologija lovca na bagove v4.0 - Recon izdanje**](https://www.youtube.com/watch?v=p4JgIu1mceI)

<figure><img src="https://github.com/carlospolop/hacktricks/blob/rs/.gitbook/assets/image%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1).png" alt=""><figcaption></figcaption></figure>

Ako vas zanima **karijera hakovanja** i hakovanje onoga što se ne može hakovati - **zapošljavamo!** (_potrebno je tečno poznavanje poljskog jezika u pisanju i govoru_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
