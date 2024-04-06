# Phishing Methodology

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Metodologija

1. Izviđajte žrtvu
2. Izaberite **domen žrtve**.
3. Izvršite osnovnu web enumeraciju **tražeći portale za prijavljivanje** koje koristi žrtva i **odlučite** koji ćete **fingirati**.
4. Koristite neke **OSINT** tehnike da **pronađete email adrese**.
5. Pripremite okruženje
6. **Kupite domen** koji ćete koristiti za procenu ribarenja
7. **Konfigurišite email servis** povezane zapise (SPF, DMARC, DKIM, rDNS)
8. Konfigurišite VPS sa **gophish**
9. Pripremite kampanju
10. Pripremite **šablon emaila**
11. Pripremite **web stranicu** za krađu pristupnih podataka
12. Pokrenite kampanju!

## Generišite slična imena domena ili kupite pouzdan domen

### Tehnike varijacije imena domena

* **Ključna reč**: Domen sadrži važnu ključnu reč originalnog domena (npr. zelster.com-management.com).
* **Podebljani poddomen**: Promenite **tačku za crticu** poddomena (npr. www-zelster.com).
* **Novi TLD**: Isto ime domena koristeći **novi TLD** (npr. zelster.org)
* **Homograft**: Zamenjuje slovo u imenu domena sa slovima koja slično izgledaju (npr. zelfser.com).
* **Transpozicija**: Menja dva slova unutar imena domena (npr. zelsetr.com).
* **Jednina/Množina**: Dodaje ili uklanja "s" na kraju imena domena (npr. zeltsers.com).
* **Izostavljanje**: Uklanja jedno slovo iz imena domena (npr. zelser.com).
* **Ponavljanje**: Ponavlja jedno slovo u imenu domena (npr. zeltsser.com).
* **Zamena**: Slično homograftu ali manje prikriveno. Zamenjuje jedno slovo u imenu domena, možda slovom u blizini originalnog slova na tastaturi (npr. zektser.com).
* **Poddomenovanje**: Uvodi **tačku** unutar imena domena (npr. ze.lster.com).
* **Umetanje**: **Umeće slovo** u ime domena (npr. zerltser.com).
* **Nedostaje tačka**: Dodajte TLD domenu. (npr. zelstercom.com)

**Automatski alati**

* [**dnstwist**](https://github.com/elceef/dnstwist)
* [**urlcrazy**](https://github.com/urbanadventurer/urlcrazy)

**Veb stranice**

* [https://dnstwist.it/](https://dnstwist.it)
* [https://dnstwister.report/](https://dnstwister.report)
* [https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/](https://www.internetmarketingninjas.com/tools/free-tools/domain-typo-generator/)

### Bitflipovanje

Postoji **mogućnost da se jedan od nekoliko bitova koji su sačuvani ili u komunikaciji automatski preokrenu** zbog različitih faktora poput sunčevih baklji, kosmičkih zraka ili hardverskih grešaka.

Kada se ovaj koncept **primeni na DNS zahteve**, moguće je da **domen koji DNS server primi** nije isti kao domen koji je prvobitno zatražen.

Na primer, jedna promena bita u domenu "windows.com" može ga promeniti u "windnws.com."

Napadači mogu **iskoristiti ovo registracijom više domena sa preokrenutim bitovima** koji su slični domenu žrtve. Njihova namera je da preusmere legitimne korisnike na svoju infrastrukturu.

Za više informacija pročitajte [https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/](https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/)

### Kupite pouzdan domen

Možete pretražiti na [https://www.expireddomains.net/](https://www.expireddomains.net) za istekli domen koji biste mogli koristiti.\
Kako biste bili sigurni da istekli domen koji ćete kupiti **već ima dobar SEO** možete proveriti kako je kategorizovan:

* [http://www.fortiguard.com/webfilter](http://www.fortiguard.com/webfilter)
* [https://urlfiltering.paloaltonetworks.com/query/](https://urlfiltering.paloaltonetworks.com/query/)

## Otkrivanje email adresa

* [https://github.com/laramies/theHarvester](https://github.com/laramies/theHarvester) (100% besplatno)
* [https://phonebook.cz/](https://phonebook.cz) (100% besplatno)
* [https://maildb.io/](https://maildb.io)
* [https://hunter.io/](https://hunter.io)
* [https://anymailfinder.com/](https://anymailfinder.com)

Kako biste **otkrili više** validnih email adresa ili **verifikovali one** koje ste već otkrili, možete proveriti da li možete da ih brute-forceujete na smtp serverima žrtve. [Saznajte kako da verifikujete/otkrijete email adresu ovde](../../network-services-pentesting/pentesting-smtp/#username-bruteforce-enumeration).\
Takođe, ne zaboravite da ako korisnici koriste **bilo koji web portal za pristup njihovim mejlovima**, možete proveriti da li je ranjiv na **brute force korisničkog imena**, i iskoristiti ranjivost ako je moguće.

## Konfigurisanje GoPhish-a

### Instalacija

Možete preuzeti sa [https://github.com/gophish/gophish/releases/tag/v0.11.0](https://github.com/gophish/gophish/releases/tag/v0.11.0)

Preuzmite i dekompresujte ga unutar `/opt/gophish` i izvršite `/opt/gophish/gophish`\
Bićete dobili lozinku za admin korisnika na portu 3333 u izlazu. Stoga, pristupite tom portu i koristite te podatke za promenu admin lozinke. Možda ćete morati da tunelujete taj port na lokalni:

```bash
ssh -L 3333:127.0.0.1:3333 <user>@<ip>
```

### Konfiguracija

**Konfiguracija TLS sertifikata**

Pre ovog koraka trebalo bi da **već kupite domen** koji ćete koristiti i on mora biti **usmeren** ka **IP adresi VPS-a** gde konfigurišete **gophish**.

```bash
DOMAIN="<domain>"
wget https://dl.eff.org/certbot-auto
chmod +x certbot-auto
sudo apt install snapd
sudo snap install core
sudo snap refresh core
sudo apt-get remove certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
certbot certonly --standalone -d "$DOMAIN"
mkdir /opt/gophish/ssl_keys
cp "/etc/letsencrypt/live/$DOMAIN/privkey.pem" /opt/gophish/ssl_keys/key.pem
cp "/etc/letsencrypt/live/$DOMAIN/fullchain.pem" /opt/gophish/ssl_keys/key.crt​
```

**Konfiguracija pošte**

Započnite instalaciju: `apt-get install postfix`

Zatim dodajte domen u sledeće datoteke:

* **/etc/postfix/virtual\_domains**
* **/etc/postfix/transport**
* **/etc/postfix/virtual\_regexp**

**Takođe promenite vrednosti sledećih promenljivih unutar /etc/postfix/main.cf**

`myhostname = <domain>`\
`mydestination = $myhostname, <domain>, localhost.com, localhost`

Na kraju izmenite datoteke **`/etc/hostname`** i **`/etc/mailname`** na ime vašeg domena i **restartujte svoj VPS.**

Sada kreirajte **DNS A zapis** za `mail.<domain>` koji pokazuje na **IP adresu** VPS-a i **DNS MX** zapis koji pokazuje na `mail.<domain>`

Sada testirajmo slanje email-a:

```bash
apt install mailutils
echo "This is the body of the email" | mail -s "This is the subject line" test@email.com
```

**Konfiguracija Gophish-a**

Zaustavite izvršavanje Gophish-a i konfigurišite ga.\
Izmenite `/opt/gophish/config.json` na sledeći način (obratite pažnju na korišćenje https):

```bash
{
"admin_server": {
"listen_url": "127.0.0.1:3333",
"use_tls": true,
"cert_path": "gophish_admin.crt",
"key_path": "gophish_admin.key"
},
"phish_server": {
"listen_url": "0.0.0.0:443",
"use_tls": true,
"cert_path": "/opt/gophish/ssl_keys/key.crt",
"key_path": "/opt/gophish/ssl_keys/key.pem"
},
"db_name": "sqlite3",
"db_path": "gophish.db",
"migrations_prefix": "db/db_",
"contact_address": "",
"logging": {
"filename": "",
"level": ""
}
}
```

**Konfigurišite gophish servis**

Da biste kreirali gophish servis kako bi se automatski pokretao i upravljao kao servis, možete kreirati fajl `/etc/init.d/gophish` sa sledećim sadržajem:

```bash
#!/bin/bash
# /etc/init.d/gophish
# initialization file for stop/start of gophish application server
#
# chkconfig: - 64 36
# description: stops/starts gophish application server
# processname:gophish
# config:/opt/gophish/config.json
# From https://github.com/gophish/gophish/issues/586

# define script variables

processName=Gophish
process=gophish
appDirectory=/opt/gophish
logfile=/var/log/gophish/gophish.log
errfile=/var/log/gophish/gophish.error

start() {
echo 'Starting '${processName}'...'
cd ${appDirectory}
nohup ./$process >>$logfile 2>>$errfile &
sleep 1
}

stop() {
echo 'Stopping '${processName}'...'
pid=$(/bin/pidof ${process})
kill ${pid}
sleep 1
}

status() {
pid=$(/bin/pidof ${process})
if [["$pid" != ""| "$pid" != "" ]]; then
echo ${processName}' is running...'
else
echo ${processName}' is not running...'
fi
}

case $1 in
start|stop|status) "$1" ;;
esac
```

Završite konfigurisanje servisa i proverite ga izvršavanjem:

```bash
mkdir /var/log/gophish
chmod +x /etc/init.d/gophish
update-rc.d gophish defaults
#Check the service
service gophish start
service gophish status
ss -l | grep "3333\|443"
service gophish stop
```

## Konfigurisanje mail servera i domena

### Sačekajte i budite legitimni

Što je stariji domen, manja je verovatnoća da će biti uhvaćen kao spam. Zato biste trebali sačekati što je duže moguće (najmanje 1 nedelju) pre phishing procene. Osim toga, ako postavite stranicu o reputacionom sektoru, reputacija koju dobijete biće bolja.

Imajte na umu da čak i ako morate da sačekate nedelju dana, možete završiti konfigurisanje svega sada.

### Konfigurisanje Reverse DNS (rDNS) zapisa

Postavite rDNS (PTR) zapis koji rešava IP adresu VPS-a u ime domena.

### SPF (Sender Policy Framework) zapis

Morate **konfigurisati SPF zapis za novi domen**. Ako ne znate šta je SPF zapis, [**pročitajte ovu stranicu**](../../network-services-pentesting/pentesting-smtp/#spf).

Možete koristiti [https://www.spfwizard.net/](https://www.spfwizard.net) da generišete svoju SPF politiku (koristite IP adresu VPS mašine)

![](<../../.gitbook/assets/image (388).png>)

Ovo je sadržaj koji treba postaviti unutar TXT zapisa unutar domena:

```bash
v=spf1 mx a ip4:ip.ip.ip.ip ?all
```

### Zapis o autentikaciji, izveštavanju i usaglašenosti poruka zasnovan na domenu (DMARC)

Morate **konfigurisati DMARC zapis za novi domen**. Ako ne znate šta je DMARC zapis, [**pročitajte ovu stranicu**](../../network-services-pentesting/pentesting-smtp/#dmarc).

Morate kreirati novi DNS TXT zapis koji pokazuje na ime hosta `_dmarc.<domen>` sa sledećim sadržajem:

```bash
v=DMARC1; p=none
```

### DomainKeys Identified Mail (DKIM)

Morate **konfigurisati DKIM za novu domenu**. Ako ne znate šta je DMARC zapis [**pročitajte ovu stranicu**](../../network-services-pentesting/pentesting-smtp/#dkim).

Ovaj tutorijal se zasniva na: [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy)

{% hint style="info" %}
Potrebno je da konkatenirate oba B64 vrednosti koje generiše DKIM ključ:

```
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0wPibdqPtzYk81njjQCrChIcHzxOp8a1wjbsoNtka2X9QXCZs+iXkvw++QsWDtdYu3q0Ofnr0Yd/TmG/Y2bBGoEgeE+YTUG2aEgw8Xx42NLJq2D1pB2lRQPW4IxefROnXu5HfKSm7dyzML1gZ1U0pR5X4IZCH0wOPhIq326QjxJZm79E1nTh3xj" "Y9N/Dt3+fVnIbMupzXE216TdFuifKM6Tl6O/axNsbswMS1TH812euno8xRpsdXJzFlB9q3VbMkVWig4P538mHolGzudEBg563vv66U8D7uuzGYxYT4WS8NVm3QBMg0QKPWZaKp+bADLkOSB9J2nUpk4Aj9KB5swIDAQAB
```
{% endhint %}

### Testirajte ocenu konfiguracije vaše e-pošte

To možete uraditi koristeći [https://www.mail-tester.com/](https://www.mail-tester.com)\
Jednostavno pristupite stranici i pošaljite e-poštu na adresu koju vam daju:

```bash
echo "This is the body of the email" | mail -s "This is the subject line" test-iimosa79z@srv1.mail-tester.com
```

Možete takođe **proveriti konfiguraciju vaše email adrese** slanjem emaila na `check-auth@verifier.port25.com` i **čitanjem odgovora** (za ovo će vam biti potrebno da **otvorite** port **25** i vidite odgovor u fajlu _/var/mail/root_ ako šaljete email kao root).\
Proverite da li prolazite sve testove:

```bash
==========================================================
Summary of Results
==========================================================
SPF check:          pass
DomainKeys check:   neutral
DKIM check:         pass
Sender-ID check:    pass
SpamAssassin check: ham
```

Možete takođe poslati **poruku na Gmail koji kontrolišete**, i proveriti **zaglavlje email-a** u svom Gmail sandučetu, `dkim=pass` treba da bude prisutan u polju zaglavlja `Authentication-Results`.

```
Authentication-Results: mx.google.com;
spf=pass (google.com: domain of contact@example.com designates --- as permitted sender) smtp.mail=contact@example.com;
dkim=pass header.i=@example.com;
```

### Uklanjanje sa Spamhouse crne liste

Stranica [www.mail-tester.com](https://www.mail-tester.com) može vam pokazati da li je vaš domen blokiran od strane Spamhouse-a. Možete zatražiti uklanjanje vašeg domena/IP adrese na: [https://www.spamhaus.org/lookup/](https://www.spamhaus.org/lookup/)

### Uklanjanje sa Microsoft crne liste

Možete zatražiti uklanjanje vašeg domena/IP adrese na [https://sender.office.com/](https://sender.office.com).

## Kreiranje i Pokretanje GoPhish Kampanje

### Profil Slanja

* Postavite **ime za identifikaciju** profila pošiljaoca
* Odlučite se iz kojeg naloga ćete slati phishing e-poštu. Predlozi: _noreply, support, servicedesk, salesforce..._
* Možete ostaviti prazna polja za korisničko ime i lozinku, ali se pobrinite da označite opciju Ignoriši greške sertifikata

![](<../../.gitbook/assets/image (253) (1) (2) (1) (1) (2) (2) (3) (3) (5) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (17).png>)

{% hint style="info" %}
Preporučuje se korišćenje funkcionalnosti "**Pošalji test e-poštu**" kako biste proverili da li sve radi.\
Preporučujem da **test e-poštu šaljete na adrese 10min mailova** kako biste izbegli da budete blokirani prilikom testiranja.
{% endhint %}

### Šablon E-pošte

* Postavite **ime za identifikaciju** šablona
* Zatim napišite **subjekat** (ništa čudno, samo nešto što biste očekivali da pročitate u redovnom e-mailu)
* Proverite da li ste označili "**Dodaj sliku za praćenje**"
* Napišite **šablon e-pošte** (možete koristiti promenljive kao u sledećem primeru):

```markup
<html>
<head>
<title></title>
</head>
<body>
<p class="MsoNormal"><span style="font-size:10.0pt;font-family:&quot;Verdana&quot;,sans-serif;color:black">Dear {{.FirstName}} {{.LastName}},</span></p>
<br />
Note: We require all user to login an a very suspicios page before the end of the week, thanks!<br />
<br />
Regards,</span></p>

WRITE HERE SOME SIGNATURE OF SOMEONE FROM THE COMPANY

<p>{{.Tracker}}</p>
</body>
</html>
```

Napomena da bi se **povećala verodostojnost e-pošte**, preporučuje se korišćenje nekog potpisa iz e-pošte klijenta. Predlozi:

* Pošaljite e-poštu na **ne postojeću adresu** i proverite da li odgovor sadrži neki potpis.
* Potražite **javne e-adrese** poput info@ex.com ili press@ex.com ili public@ex.com i pošaljite im e-poštu i sačekajte odgovor.
* Pokušajte da kontaktirate **neku validnu otkrivenu** e-adresu i sačekajte odgovor

![](<../../.gitbook/assets/image (393).png>)

{% hint style="info" %}
Šablon e-pošte takođe omogućava da se **prilože datoteke za slanje**. Ako želite da ukradete NTLM izazove koristeći neke posebno izrađene datoteke/dokumente [pročitajte ovu stranicu](../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md).
{% endhint %}

### Landing stranica

* Napišite **ime**
* **Napišite HTML kod** web stranice. Imajte na umu da možete **uvoziti** web stranice.
* Označite **Zabeleži podatke koji su poslati** i **Zabeleži lozinke**
* Postavite **preusmerenje**

![](<../../.gitbook/assets/image (394).png>)

{% hint style="info" %}
Obično ćete morati da izmenite HTML kod stranice i obavite neke testove lokalno (možda koristeći neki Apache server) **dok ne budete zadovoljni rezultatima**. Zatim, napišite taj HTML kod u okviru.\
Imajte na umu da ako trebate **koristiti neke statičke resurse** za HTML (možda neke CSS i JS stranice) možete ih sačuvati u _**/opt/gophish/static/endpoint**_ i zatim im pristupiti sa _**/static/\<ime datoteke>**_
{% endhint %}

{% hint style="info" %}
Za preusmerenje možete **preusmeriti korisnike na legitimnu glavnu web stranicu** žrtve, ili ih preusmeriti na _/static/migration.html_ na primer, staviti neki **vrtlog (**[**https://loading.io/**](https://loading.io)**) na 5 sekundi, a zatim naznačiti da je proces bio uspešan**.
{% endhint %}

### Korisnici i grupe

* Postavite ime
* **Uvezite podatke** (imajte na umu da za korišćenje šablona za primer trebate ime, prezime i e-adresu svakog korisnika)

![](<../../.gitbook/assets/image (395).png>)

### Kampanja

Na kraju, kreirajte kampanju odabirom imena, šablona e-pošte, landing stranice, URL-a, profila slanja i grupe. Imajte na umu da će URL biti link poslat žrtvama

Imajte na umu da **Profil slanja omogućava slanje test e-pošte da biste videli kako će konačna phishing e-pošta izgledati**:

![](<../../.gitbook/assets/image (396).png>)

{% hint style="info" %}
Preporučujem da **test e-poštu šaljete na adrese 10min mailova** kako biste izbegli da budete na crnoj listi praveći testove.
{% endhint %}

Kada je sve spremno, samo pokrenite kampanju!

## Kloniranje veb sajta

Ako iz nekog razloga želite da klonirate veb sajt, pogledajte sledeću stranicu:

{% content-ref url="clone-a-website.md" %}
[clone-a-website.md](clone-a-website.md)
{% endcontent-ref %}

## Dokumenti i fajlovi sa zadnjim vratima

U nekim phishing procenama (uglavnom za Crvene timove) takođe ćete želeti da **pošaljete fajlove koji sadrže neku vrstu zadnjeg vrata** (možda C2 ili možda nešto što će pokrenuti autentifikaciju).\
Pogledajte sledeću stranicu za neke primere:

{% content-ref url="phishing-documents.md" %}
[phishing-documents.md](phishing-documents.md)
{% endcontent-ref %}

## Phishing MFA

### Putem Proxy MitM

Prethodni napad je prilično lukav jer lažirate pravu veb lokaciju i prikupljate informacije postavljene od strane korisnika. Nažalost, ako korisnik nije uneo tačnu lozinku ili ako je aplikacija koju ste lažirali konfigurisana sa 2FA, **te informacije vam neće dozvoliti da se predstavite kao prevareni korisnik**.

Tu su korisni alati poput [**evilginx2**](https://github.com/kgretzky/evilginx2)**,** [**CredSniper**](https://github.com/ustayready/CredSniper) i [**muraena**](https://github.com/muraenateam/muraena). Ovaj alat će vam omogućiti da generišete napad poput MitM-a. Osnovno, napadi funkcionišu na sledeći način:

1. **Predstavljate se kao obrazac za prijavu** prave veb stranice.
2. Korisnik **šalje** svoje **poverljive podatke** na vašu lažnu stranicu, a alat ih šalje na pravu veb stranicu, **proveravajući da li podaci funkcionišu**.
3. Ako je nalog konfigurisan sa **2FA**, stranica MitM će zatražiti to, a kada **korisnik unese** to, alat će ga poslati na pravu veb stranicu.
4. Kada se korisnik autentifikuje, vi (kao napadač) ćete imati **zabeležene poverljive podatke, 2FA, kolačiće i sve informacije** o svakoj interakciji dok alat vrši MitM.

### Putem VNC-a

Šta ako umesto **slanja žrtve na zlonamernu stranicu** sa istim izgledom kao originalna, pošaljete je na **VNC sesiju sa pregledačem povezanim sa pravom veb stranicom**? Bićete u mogućnosti da vidite šta radi, ukradete lozinku, korišćeni MFA, kolačiće...\
To možete uraditi sa [**EvilnVNC**](https://github.com/JoelGMSec/EvilnoVNC)

## Otkrivanje detekcije

Očigledno je jedan od najboljih načina da saznate da li ste otkriveni da **pretražite svoj domen u crnim listama**. Ako se pojavi na listi, na neki način vaš domen je otkriven kao sumnjiv.\
Jednostavan način da proverite da li se vaš domen pojavljuje na bilo kojoj crnoj listi je da koristite [https://malwareworld.com/](https://malwareworld.com)

Međutim, postoje i drugi načini da saznate da li žrtva **aktivno traži sumnjive phishing aktivnosti u divljini** kako je objašnjeno u:

{% content-ref url="detecting-phising.md" %}
[detecting-phising.md](detecting-phising.md)
{% endcontent-ref %}

Možete **kupiti domen sa veoma sličnim imenom** domenu žrtve **i/ili generisati sertifikat** za **poddomen** domena koji kontrolišete **sadržeći** ključnu reč **imenom** domena žrtve. Ako **žrtva** izvrši bilo kakvu vrstu **DNS ili HTTP interakcije** sa njima, saznaćete da **aktivno traži** sumnjive domene i moraćete biti veoma prikriveni.

### Procenite phishing

Koristite [**Phishious** ](https://github.com/Rices/Phishious)da procenite da li će vaša e-pošta završiti u spam folderu ili da li će biti blokirana ili uspešna.

## Reference

* [https://zeltser.com/domain-name-variations-in-phishing/](https://zeltser.com/domain-name-variations-in-phishing/)
* [https://0xpatrik.com/phishing-domains/](https://0xpatrik.com/phishing-domains/)
* [https://darkbyte.net/robando-sesiones-y-bypasseando-2fa-con-evilnovnc/](https://darkbyte.net/robando-sesiones-y-bypasseando-2fa-con-evilnovnc/)
* [https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
