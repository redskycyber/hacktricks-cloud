# macOS Office Sandbox Bypasses

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### Bypass Sandbox-a u Word-u putem Launch Agents

Aplikacija koristi **prilagođeni Sandbox** koristeći privilegiju **`com.apple.security.temporary-exception.sbpl`** i ovaj prilagođeni sandbox omogućava pisanje fajlova bilo gde, sve dok ime fajla počinje sa `~$`: `(require-any (require-all (vnode-type REGULAR-FILE) (regex #"(^|/)~$[^/]+$")))`

Stoga, izbegavanje je bilo jednostavno kao **pisanje `plist`** LaunchAgent-a u `~/Library/LaunchAgents/~$escape.plist`.

Pogledajte [**originalni izveštaj ovde**](https://www.mdsec.co.uk/2018/08/escaping-the-sandbox-microsoft-office-on-macos/).

### Bypass Sandbox-a u Word-u putem Login stavki i zip-a

Zapamtite da od prvog bekstva, Word može pisati proizvoljne fajlove čije ime počinje sa `~$`, iako nakon zakrpe prethodne ranjivosti nije bilo moguće pisati u `/Library/Application Scripts` ili u `/Library/LaunchAgents`.

Otkriveno je da je iz sandbox-a moguće kreirati **Login stavku** (aplikacije koje će se izvršiti kada se korisnik prijavi). Međutim, ove aplikacije **neće se izvršiti osim ako** nisu **notarizovane** i nije moguće dodati argumente (tako da ne možete pokrenuti reverznu ljusku koristeći **`bash`**).

Od prethodnog bypass-a Sandbox-a, Microsoft je onemogućio opciju pisanja fajlova u `~/Library/LaunchAgents`. Međutim, otkriveno je da ako stavite **zip fajl kao Login stavku**, `Archive Utility` će ga samo **otpakovati** na trenutnoj lokaciji. Dakle, pošto podrazumevano folder `LaunchAgents` iz `~/Library` nije kreiran, bilo je moguće **zapakovati plist u `LaunchAgents/~$escape.plist`** i **postaviti** zip fajl u **`~/Library`** tako da će prilikom dekompresije stići do odredišta trajnosti.

Pogledajte [**originalni izveštaj ovde**](https://objective-see.org/blog/blog\_0x4B.html).

### Bypass Sandbox-a u Word-u putem Login stavki i .zshenv

(Zapamtite da od prvog bekstva, Word može pisati proizvoljne fajlove čije ime počinje sa `~$`).

Međutim, prethodna tehnika je imala ograničenje, ako folder **`~/Library/LaunchAgents`** postoji jer ga je neki drugi softver kreirao, ona bi propala. Zato je otkrivena druga Login stavka za ovo.

Napadač bi mogao kreirati fajlove **`.bash_profile`** i **`.zshenv`** sa payload-om za izvršavanje, a zatim ih zapakovati i **zapisati zip u korisnički folder** žrtve: **`~/~$escape.zip`**.

Zatim, dodajte zip fajl u **Login stavke**, a zatim i **aplikaciju Terminal**. Kada se korisnik ponovo prijavi, zip fajl će biti dekompresovan u korisnički fajl, prebrisavajući **`.bash_profile`** i **`.zshenv`** i stoga će terminal izvršiti jedan od ovih fajlova (zavisno od toga da li se koristi bash ili zsh).

Pogledajte [**originalni izveštaj ovde**](https://desi-jarvis.medium.com/office365-macos-sandbox-escape-fcce4fa4123c).

### Bypass Sandbox-a u Word-u sa Open i env promenljivama

Iz sandbox procesa i dalje je moguće pozvati druge procese koristeći alatku **`open`**. Osim toga, ovi procesi će se izvršavati **unutar svog sopstvenog sandbox-a**.

Otkriveno je da open alatka ima opciju **`--env`** za pokretanje aplikacije sa **specifičnim env promenljivama**. Stoga je bilo moguće kreirati **`.zshenv` fajl** unutar foldera **unutar** sandbox-a i koristiti `open` sa `--env` postavljajući **`HOME` promenljivu** na taj folder otvarajući tu `Terminal` aplikaciju, koja će izvršiti `.zshenv` fajl (iz nekog razloga bilo je potrebno i postaviti promenljivu `__OSINSTALL_ENVIROMENT`).

Pogledajte [**originalni izveštaj ovde**](https://perception-point.io/blog/technical-analysis-of-cve-2021-30864/).

### Bypass Sandbox-a u Word-u sa Open i stdin

Alatka **`open`** takođe podržava parametar **`--stdin`** (a nakon prethodnog bypass-a više nije bilo moguće koristiti `--env`).

Stvar je u tome da čak i ako je **`python`** potpisan od strane Apple-a, **neće izvršiti** skriptu sa **`quarantine`** atributom. Međutim, bilo je moguće proslediti mu skriptu putem stdin-a tako da neće proveravati da li je karantinirana ili ne:

1. Ubacite fajl **`~$exploit.py`** sa proizvoljnim Python komandama.
2. Pokrenite _open_ **`–stdin='~$exploit.py' -a Python`**, što pokreće Python aplikaciju sa našim ubačenim fajlom kao standardni ulaz. Python srećno izvršava naš kod i pošto je to podproces _launchd_-a, nije vezan za Word-ova pravila sandbox-a.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na \*\*Twitter

</details>
