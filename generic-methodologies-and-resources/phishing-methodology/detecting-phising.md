# Detecting Phishing

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Uvod

Da biste otkrili pokušaj fišinga, važno je **razumeti tehnike fišinga koje se danas koriste**. Na roditeljskoj stranici ovog posta možete pronaći te informacije, pa ako niste upoznati sa tehnikama koje se danas koriste, preporučujem vam da odete na roditeljsku stranicu i pročitate barem taj deo.

Ovaj post se zasniva na ideji da će **napadači pokušati na neki način da imitiraju ili koriste ime domena žrtve**. Ako je vaš domen nazvan `primer.com` i fišing napad se izvrši koristeći potpuno drugačije ime domena kao što je `osvojiliste.com`, ove tehnike neće otkriti takav napad.

## Varijacije imena domena

Prilično je **lako** otkriti one pokušaje **fišinga** koji koriste **slično ime domena** u e-mailu.\
Dovoljno je **generisati listu najverovatnijih imena za fišing** koje napadač može koristiti i **proveriti** da li je **registrovano** ili samo proveriti da li postoji neka **IP adresa** koja je koristi.

### Pronalaženje sumnjivih domena

Za tu svrhu možete koristiti bilo koji od sledećih alata. Imajte na umu da će ovi alati automatski izvršiti DNS zahtev kako bi proverili da li je domen dodeljen nekoj IP adresi:

* [**dnstwist**](https://github.com/elceef/dnstwist)
* [**urlcrazy**](https://github.com/urbanadventurer/urlcrazy)

### Bitflipping

**Kratak opis ove tehnike možete pronaći na roditeljskoj stranici. Ili pročitajte originalno istraživanje na** [**https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/**](https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/)

Na primer, jedna promena bita u domenu microsoft.com može ga pretvoriti u _windnws.com._\
**Napadači mogu registrovati što više domena sa promenjenim bitovima kako bi preusmerili legitimne korisnike na svoju infrastrukturu**.

**Svi mogući domeni sa promenjenim bitovima takođe bi trebalo da se prate.**

### Osnovne provere

Kada imate listu potencijalno sumnjivih imena domena, trebali biste ih **proveriti** (pre svega portove HTTP i HTTPS) da biste **videli da li koriste neki sličan obrazac za prijavu** kao neki od domena žrtve.\
Takođe možete proveriti port 3333 da biste videli da li je otvoren i pokreće instancu `gophish`.\
Takođe je interesantno znati **koliko je star svaki otkriveni sumnjivi domen**, što je mlađi, to je rizičniji.\
Možete takođe dobiti **screenshot-ove** HTTP i/ili HTTPS sumnjivih web stranica da biste videli da li su sumnjive i u tom slučaju **pristupiti im da biste detaljnije istražili**.

### Napredne provere

Ako želite da odete korak dalje, preporučujem vam da **pratite te sumnjive domene i povremeno tražite nove** (svaki dan? to traje samo nekoliko sekundi/minuta). Takođe biste trebali **proveriti** otvorene **porteve** povezane sa IP adresama i **tražiti instance `gophish` ili sličnih alata** (da, napadači takođe prave greške) i **pratiti HTTP i HTTPS web stranice sumnjivih domena i poddomena** da biste videli da li su kopirali neki obrazac za prijavu sa web stranica žrtve.\
Da biste **automatizovali ovo**, preporučujem da imate listu obrazaca za prijavu domena žrtve, pretražite sumnjive web stranice i uporedite svaki pronađeni obrazac za prijavu unutar sumnjivih domena sa svakim obrascem za prijavu domena žrtve koristeći nešto poput `ssdeep`.\
Ako ste locirali obrasce za prijavu sumnjivih domena, možete pokušati da **pošaljete lažne podatke za prijavu** i **proverite da li vas preusmerava na domen žrtve**.

## Imena domena sa ključnim rečima

Roditeljska stranica takođe pominje tehniku varijacije imena domena koja se sastoji od stavljanja **imenom domena žrtve unutar većeg domena** (npr. paypal-financial.com za paypal.com).

### Transparentnost sertifikata

Nije moguće primeniti prethodni "Brute-Force" pristup, ali je zapravo **moguće otkriti takve pokušaje fišinga** zahvaljujući transparentnosti sertifikata. Svaki put kada sertifikat izda CA, detalji postaju javni. To znači da čitanjem transparentnosti sertifikata ili čak praćenjem iste, **moguće je pronaći domene koji koriste ključnu reč u svom imenu**. Na primer, ako napadač generiše sertifikat za [https://paypal-financial.com](https://paypal-financial.com), čitanjem sertifikata je moguće pronaći ključnu reč "paypal" i saznati da se koristi sumnjiva e-mail adresa.

Post [https://0xpatrik.com/phishing-domains/](https://0xpatrik.com/phishing-domains/) predlaže da možete koristiti Censys za pretragu sertifikata koji utiču na određenu ključnu reč i filtrirati ih po datumu (samo "novi" sertifikati) i po izdavaču CA "Let's Encrypt":

![https://0xpatrik.com/content/images/2018/07/cert\_listing.png](<../../.gitbook/assets/image (390).png>)

Međutim, možete uraditi "isto" koristeći besplatnu veb stranicu [**crt.sh**](https://crt.sh). Možete **pretraživati po ključnoj reči** i **filtrirati rezultate** po datumu i CA ako želite.

![](<../../.gitbook/assets/image (391).png>)

Koristeći ovu poslednju opciju, čak možete koristiti polje Matching Identities da biste videli da li se bilo koja identifikacija sa pravog domena poklapa sa nekim od sumnjivih domena (imajte na umu da sumnjiv domen može biti lažno pozitivan).

**Još jedna alternativa** je fantastični projekat koji se zove [**CertStream**](https://medium.com/cali-dog-security/introducing-certstream-3fc13bb98067). CertStream pruža real-time tok novo generisanih sertifikata koje možete koristiti da biste u (skoro) realnom vremenu otkrili određene ključne reči

### **Novi domeni**

**Još jedna alternativa** je da prikupite listu **nedavno registrovanih domena** za neke TLD-ove ([Whoxy](https://www.whoxy.com/newly-registered-domains/) pruža takvu uslugu) i **proverite ključne reči u tim domenima**. Međutim, dugi domeni obično koriste jedan ili više poddomena, pa ključna reč neće biti vidljiva unutar FLD-a i nećete moći pronaći poddomenu za phishing.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
