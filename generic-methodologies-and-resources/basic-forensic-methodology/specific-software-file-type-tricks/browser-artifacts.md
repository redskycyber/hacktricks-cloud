# Browser Artifacts

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako kreirate i **automatizujete radne tokove** uz pomoć najnaprednijih alata zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Artifakti pregledača <a href="#id-3def" id="id-3def"></a>

Artifakti pregledača uključuju različite vrste podataka koje čuvaju veb pregledači, poput istorije navigacije, obeleživača i keš podataka. Ovi artifakti se čuvaju u specifičnim fasciklama unutar operativnog sistema, razlikujući se po lokaciji i imenu u različitim pregledačima, ali uglavnom čuvajući slične vrste podataka.

Evo sažetka najčešćih artifakata pregledača:

* **Istorija navigacije**: Prati posete korisnika veb sajtovima, korisno za identifikaciju poseta zlonamernim sajtovima.
* **Automatski podaci**: Predlozi zasnovani na čestim pretragama, pružajući uvide kada se kombinuju sa istorijom navigacije.
* **Obeleživači**: Sajtovi koje je korisnik sačuvao radi brzog pristupa.
* **Proširenja i dodaci**: Proširenja pregledača ili dodaci instalirani od strane korisnika.
* **Keš**: Čuva veb sadržaj (npr. slike, JavaScript fajlove) radi poboljšanja vremena učitavanja sajtova, vredno za forenzičku analizu.
* **Prijave**: Sačuvane prijavne informacije.
* **Favikoni**: Ikone povezane sa sajtovima, prikazuju se u tabovima i obeleživačima, korisne za dodatne informacije o posetama korisnika.
* **Sesije pregledača**: Podaci vezani za otvorene sesije pregledača.
* **Preuzimanja**: Zapisi fajlova preuzetih putem pregledača.
* **Podaci o obrascima**: Informacije unete u veb obrasce, sačuvane za buduće predloge automatskog popunjavanja.
* **Sličice**: Pregledne slike veb sajtova.
* **Prilagođeni rečnik.txt**: Reči dodate od strane korisnika u rečnik pregledača.

## Firefox

Firefox organizuje korisničke podatke unutar profila, čuvajući ih na specifičnim lokacijama zavisno od operativnog sistema:

* **Linux**: `~/.mozilla/firefox/`
* **MacOS**: `/Users/$USER/Library/Application Support/Firefox/Profiles/`
* **Windows**: `%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\`

Fajl `profiles.ini` unutar ovih direktorijuma navodi korisničke profile. Podaci svakog profila se čuvaju u fascikli nazvanoj prema promenljivoj `Path` unutar `profiles.ini`, smeštenoj u istom direktorijumu kao i sam `profiles.ini`. Ako nedostaje fascikla profila, možda je obrisana.

Unutar svake fascikle profila, možete pronaći nekoliko važnih fajlova:

* **places.sqlite**: Čuva istoriju, obeleživače i preuzimanja. Alati poput [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) na Windows-u mogu pristupiti podacima istorije.
* Koristite specifične SQL upite za izvlačenje informacija o istoriji i preuzimanjima.
* **bookmarkbackups**: Sadrži rezervne kopije obeleživača.
* **formhistory.sqlite**: Čuva podatke o veb obrascima.
* **handlers.json**: Upravljači protokolima.
* **persdict.dat**: Prilagođene reči rečnika.
* **addons.json** i **extensions.sqlite**: Informacije o instaliranim dodacima i proširenjima.
* **cookies.sqlite**: Čuvanje kolačića, sa [MZCookiesView](https://www.nirsoft.net/utils/mzcv.html) dostupnim za inspekciju na Windows-u.
* **cache2/entries** ili **startupCache**: Keš podaci, pristupačni putem alata poput [MozillaCacheView](https://www.nirsoft.net/utils/mozilla\_cache\_viewer.html).
* **favicons.sqlite**: Čuva favikone.
* **prefs.js**: Korisnička podešavanja i preferencije.
* **downloads.sqlite**: Starija baza preuzimanja, sada integrisana u places.sqlite.
* **thumbnails**: Slike pregleda veb sajtova.
* **logins.json**: Šifrovane prijavne informacije.
* **key4.db** ili **key3.db**: Čuva ključeve za šifrovanje osetljivih informacija.

Dodatno, provera anti-phishing podešavanja pregledača može se obaviti pretragom unosa `browser.safebrowsing` u `prefs.js`, što ukazuje da li su funkcije sigurnog pregledanja omogućene ili onemogućene.

Za pokušaj dešifrovanja glavne lozinke, možete koristiti [https://github.com/unode/firefox\_decrypt](https://github.com/unode/firefox\_decrypt)\
Pomoću sledećeg skripta i poziva možete specificirati fajl sa lozinkom za brute force:

{% code title="brute.sh" %}
```
```
{% endcode %}

```bash
#!/bin/bash

#./brute.sh top-passwords.txt 2>/dev/null | grep -A2 -B2 "chrome:"
passfile=$1
while read pass; do
echo "Trying $pass"
echo "$pass" | python firefox_decrypt.py
done < $passfile
```

## Google Chrome

Google Chrome čuva korisničke profile na specifičnim lokacijama zavisno od operativnog sistema:

* **Linux**: `~/.config/google-chrome/`
* **Windows**: `C:\Users\XXX\AppData\Local\Google\Chrome\User Data\`
* **MacOS**: `/Users/$USER/Library/Application Support/Google/Chrome/`

U ovim direktorijumima, većina korisničkih podataka se može pronaći u fasciklama **Default/** ili **ChromeDefaultData/**. Sledeći fajlovi sadrže značajne podatke:

* **History**: Sadrži URL-ove, preuzimanja i ključne reči pretrage. Na Windows-u, [ChromeHistoryView](https://www.nirsoft.net/utils/chrome\_history\_view.html) se može koristiti za čitanje istorije. Kolona "Transition Type" ima različita značenja, uključujući klikove korisnika na linkove, unete URL-ove, podnesene forme i osvežavanja stranica.
* **Cookies**: Čuva kolačiće. Za inspekciju, dostupan je [ChromeCookiesView](https://www.nirsoft.net/utils/chrome\_cookies\_view.html).
* **Cache**: Čuva keširane podatke. Windows korisnici mogu koristiti [ChromeCacheView](https://www.nirsoft.net/utils/chrome\_cache\_view.html) za inspekciju.
* **Bookmarks**: Korisnički obeleživači.
* **Web Data**: Sadrži istoriju formi.
* **Favicons**: Čuva favikone veb sajtova.
* **Login Data**: Uključuje podatke za prijavljivanje poput korisničkih imena i lozinki.
* **Current Session**/**Current Tabs**: Podaci o trenutnoj sesiji pregledanja i otvorenim tabovima.
* **Last Session**/**Last Tabs**: Informacije o sajtovima aktivnim tokom poslednje sesije pre nego što je Chrome zatvoren.
* **Extensions**: Direktorijumi za proširenja i dodatke pregledača.
* **Thumbnails**: Čuva sličice veb sajtova.
* **Preferences**: Fajl bogat informacijama, uključujući podešavanja za dodatke, proširenja, iskačuće prozore, obaveštenja i više.
* **Ugrađena anti-phishing za pregledač**: Da biste proverili da li je anti-phishing i zaštita od malvera omogućena, pokrenite `grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`. Potražite `{"enabled: true,"}` u izlazu.

## **Obnova podataka iz SQLite baze podataka**

Kao što možete primetiti u prethodnim sekcijama, kako Chrome tako i Firefox koriste **SQLite** baze podataka za čuvanje podataka. Moguće je **obnoviti obrisane unose korišćenjem alata** [**sqlparse**](https://github.com/padfoot999/sqlparse) **ili** [**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases).

## **Internet Explorer 11**

Internet Explorer 11 upravlja svojim podacima i metapodacima na različitim lokacijama, pomažući u razdvajanju čuvanih informacija i odgovarajućih detalja radi lakšeg pristupa i upravljanja.

### Čuvanje metapodataka

Metapodaci za Internet Explorer se čuvaju u `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data` (sa VX koji može biti V01, V16 ili V24). Pored toga, fajl `V01.log` može pokazati razlike u vremenu modifikacije u odnosu na `WebcacheVX.data`, što ukazuje na potrebu popravke korišćenjem `esentutl /r V01 /d`. Ovi metapodaci, smešteni u ESE bazi podataka, mogu biti obnovljeni i inspektovani korišćenjem alata poput photorec i [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html), redom. Unutar tabele **Containers**, može se razlikovati specifične tabele ili kontejneri gde je smešten svaki segment podataka, uključujući detalje keša za druge Microsoft alate poput Skype-a.

### Inspekcija keša

Alat [IECacheView](https://www.nirsoft.net/utils/ie\_cache\_viewer.html) omogućava inspekciju keša, zahtevajući lokaciju fascikle za ekstrakciju keš podataka. Metapodaci za keš uključuju ime fajla, direktorijum, broj pristupa, poreklo URL-a i vremenske oznake koje pokazuju vreme kreiranja, pristupa, modifikacije i isteka keša.

### Upravljanje kolačićima

Kolačiće možete istražiti korišćenjem [IECookiesView](https://www.nirsoft.net/utils/iecookies.html), pri čemu metapodaci obuhvataju imena, URL-ove, broj pristupa i različite detalje vezane za vreme. Trajni kolačići se čuvaju u `%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies`, dok se sesijski kolačići nalaze u memoriji.

### Detalji preuzimanja

Metapodaci o preuzimanjima su dostupni putem [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html), pri čemu specifični kontejneri čuvaju podatke poput URL-a, tipa fajla i lokacije preuzimanja. Fizički fajlovi se mogu pronaći pod `%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory`.

### Istorija pregledanja

Za pregled istorije pregledanja, može se koristiti [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html), zahtevajući lokaciju izdvojenih fajlova istorije i konfiguraciju za Internet Explorer. Metapodaci ovde uključuju vreme modifikacije i pristupa, zajedno sa brojem pristupa. Fajlovi istorije se nalaze u `%userprofile%\Appdata\Local\Microsoft\Windows\History`.

### Uneti URL-ovi

Uneti URL-ovi i vremena njihovog korišćenja se čuvaju u registru pod `NTUSER.DAT` na `Software\Microsoft\InternetExplorer\TypedURLs` i `Software\Microsoft\InternetExplorer\TypedURLsTime`, prateći poslednjih 50 URL-ova unetih od strane korisnika i njihova poslednja vremena unosa.

## Microsoft Edge

Microsoft Edge čuva korisničke podatke u `%userprofile%\Appdata\Local\Packages`. Putanje za različite tipove podataka su:

* **Putanja profila**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC`
* **Istorija, Kolačići i Preuzimanja**: `C:\Users\XX\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat`
* **Podešavanja, Obeleživači i Lista za čitanje**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb`
* **Keš**: `C:\Users\XXX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC#!XXX\MicrosoftEdge\Cache`
* **Poslednje aktivne sesije**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active`

## Safari

Podaci Safari pregledača se čuvaju na lokaciji `/Users/$User/Library/Safari`. Ključni fajlovi uključuju:

* **History.db**: Sadrži tabele `history_visits` i `history_items` sa URL-ovima i vremenima poseta. Koristite `sqlite3` za upite.
* **Downloads.plist**: Informacije o preuzetim fajlovima.
* **Bookmarks.plist**: Čuva URL-ove obeleženih stranica.
* **TopSites.plist**: Najposećeniji sajtovi.
* **Extensions.plist**: Lista proširenja Safari pregledača. Koristite `plutil` ili `pluginkit` za dobijanje.
* **UserNotificationPermissions.plist**: Domeni kojima je dozvoljeno slanje obaveštenja. Koristite `plutil` za parsiranje.
* **LastSession.plist**: Tabovi iz poslednje sesije. Koristite `plutil` za parsiranje.
* **Ugrađena anti-phishing za pregledač**: Proverite korišćenjem `defaults read com.apple.Safari WarnAboutFraudulentWebsites`. Odgovor 1 ukazuje na aktivnu funkciju.

## Opera

Podaci Operinog pregledača se nalaze u `/Users/$USER/Library/Application Support/com.operasoftware.Opera` i dele format za istoriju i preuzimanja sa Chrome-om.

* **Ugrađena anti-phishing za pregledač**: Proverite da li je `fraud_protection_enabled` u fajlu Preferences postavljen na `true` korišćenjem `grep`.

Ove putanje i komande su ključne za pristupanje i razumevanje podataka o pregledanju čuvanih od strane različitih veb pregledača.

## Reference

* [https://nasbench.medium.com/web-browsers-forensics-7e99940c579a](https://nasbench.medium.com/web-browsers-forensics-7e99940c579a)
* [https://www.sentinelone.com/labs/macos-incident-response-part-3-system-manipulation/](https://www.sentinelone.com/labs/macos-incident-response-part-3-system-manipulation/)
* [https://books.google.com/books?id=jfMqCgAAQBAJ\&pg=PA128\&lpg=PA128\&dq=%22This+file](https://books.google.com/books?id=jfMqCgAAQBAJ\&pg=PA128\&lpg=PA128\&dq=%22This+file)

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako kreirate i **automatizujete radne tokove** uz pomoć najnaprednijih alata zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od početnika do stručnjaka sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

\* Ako želite da vidite svoju \*\*kompaniju reklamiranu na HackTricks\*\* ili \*\*preuzmete HackTricks u PDF formatu\*\* proverite \[\*\*PLANOVE ZA PRIJAVU\*\*]\(https://github.com/sponsors/carlospolop)! \* Nabavite \[\*\*zvanični PEASS & HackTricks suvenir\*\*]\(https://peass.creator-spring.com) \* Otkrijte \[\*\*Porodicu PEASS\*\*]\(https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih \[\*\*NFT-ova\*\*]\(https://opensea.io/collection/the-peass-family) \* \*\*Pridružite se\*\* 💬 \[\*\*Discord grupi\*\*]\(https://discord.gg/hRep4RUj7f) ili \[\*\*telegram grupi\*\*]\(https://t.me/peass) ili nas \*\*pratite\*\* na \*\*Twitteru\*\* 🐦 \[\*\*@hacktricks\\\_live\*\*]\(https://twitter.com/hacktricks\\\_live)\*\*.\*\* \* \*\*Podelite svoje hakovanje trikove slanjem PR-ova na\*\* \[\*\*HackTricks\*\*]\(https://github.com/carlospolop/hacktricks) i \[\*\*HackTricks Cloud\*\*]\(https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
