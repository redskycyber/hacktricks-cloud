# macOS Security Protections

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Gatekeeper

Gatekeeper se obično koristi za kombinaciju **Quarantine + Gatekeeper + XProtect**, 3 macOS sigurnosnih modula koji će pokušati **da spreče korisnike da izvrše potencijalno zlonamerni softver koji je preuzet**.

Više informacija u:

{% content-ref url="macos-gatekeeper.md" %}
[macos-gatekeeper.md](macos-gatekeeper.md)
{% endcontent-ref %}

## Ograničenja procesa

### SIP - Sistem zaštite integriteta

{% content-ref url="macos-sip.md" %}
[macos-sip.md](macos-sip.md)
{% endcontent-ref %}

### Pesak

MacOS pesak **ograničava aplikacije** koje se izvršavaju unutar peska na **dozvoljene radnje navedene u profilu peska** sa kojim aplikacija radi. Ovo pomaže da se osigura da **aplikacija pristupa samo očekivanim resursima**.

{% content-ref url="macos-sandbox/" %}
[macos-sandbox](macos-sandbox/)
{% endcontent-ref %}

### TCC - **Transparentnost, saglasnost i kontrola**

**TCC (Transparentnost, saglasnost i kontrola)** je sigurnosni okvir. Namjenjen je **upravljanju dozvolama** aplikacija, posebno regulisanjem njihovog pristupa osetljivim funkcijama. To uključuje elemente kao što su **usluge lokacije, kontakti, fotografije, mikrofon, kamera, pristupačnost i pun pristup disku**. TCC osigurava da aplikacije mogu pristupiti ovim funkcijama samo nakon dobijanja izričite saglasnosti korisnika, čime se jača privatnost i kontrola nad ličnim podacima.

{% content-ref url="macos-tcc/" %}
[macos-tcc](macos-tcc/)
{% endcontent-ref %}

### Ograničenja pokretanja/okruženja i keš poverenja

Ograničenja pokretanja u macOS-u su sigurnosna funkcija koja **reguliše pokretanje procesa** definisanjem **ko može pokrenuti** proces, **kako** i **odakle**. Uvedena u macOS Ventura, kategorizuje sistemski binarni kod u kategorije ograničenja unutar **keša poverenja**. Svaki izvršni binarni kod ima postavljena **pravila** za njegovo **pokretanje**, uključujući **samo**, **roditelj** i **odgovorni** ograničenja. Prošireno na aplikacije trećih strana kao **Ograničenja okruženja** u macOS Sonoma, ove funkcije pomažu u ublažavanju potencijalnih zloupotreba sistema regulisanjem uslova pokretanja procesa.

{% content-ref url="macos-launch-environment-constraints.md" %}
[macos-launch-environment-constraints.md](macos-launch-environment-constraints.md)
{% endcontent-ref %}

## MRT - Alat za uklanjanje malvera

Alat za uklanjanje malvera (MRT) je još jedan deo sigurnosne infrastrukture macOS-a. Kao što naziv sugeriše, glavna funkcija MRT-a je **uklanjanje poznatih malvera sa zaraženih sistema**.

Kada se malver otkrije na Mac-u (bilo putem XProtect-a ili na neki drugi način), MRT se može koristiti za automatsko **uklanjanje malvera**. MRT radi tiho u pozadini i obično se pokreće kada se sistem ažurira ili kada se preuzme nova definicija malvera (izgleda da su pravila koja MRT koristi za otkrivanje malvera unutar binarnog koda).

Iako su i XProtect i MRT deo sigurnosnih mera macOS-a, obavljaju različite funkcije:

* **XProtect** je preventivni alat. **Proverava datoteke prilikom preuzimanja** (putem određenih aplikacija) i ako otkrije bilo koji poznati tip malvera, **sprečava otvaranje datoteke**, čime sprečava infekciju sistema malverom.
* **MRT**, s druge strane, je **reaktivni alat**. Radi nakon što je malver otkriven na sistemu, sa ciljem uklanjanja zlonamernog softvera radi čišćenja sistema.

Aplikacija MRT se nalazi u **`/Library/Apple/System/Library/CoreServices/MRT.app`**

## Upravljanje pozadinskim zadacima

**macOS** sada **upozorava** svaki put kada alat koristi dobro poznatu **tehniku za trajno izvršavanje koda** (kao što su stavke za prijavljivanje, daemoni...), tako da korisnik bolje zna **koji softver je trajno prisutan**.

<figure><img src="../../../.gitbook/assets/image (711).png" alt=""><figcaption></figcaption></figure>

Ovo se pokreće sa **demonom** koji se nalazi u `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Versions/A/Resources/backgroundtaskmanagementd` i **agentom** u `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Support/BackgroundTaskManagementAgent.app`

Način na koji **`backgroundtaskmanagementd`** zna da je nešto instalirano u trajnom folderu je **dobijanje FSEvents** i kreiranje nekih **handlera** za njih.

Osim toga, postoji plist datoteka koja sadrži **dobro poznate aplikacije** koje često ostaju prisutne, održavane od strane Apple-a, a nalazi se u: `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Versions/A/Resources/attributions.plist`

```json
[...]
"us.zoom.ZoomDaemon" => {
"AssociatedBundleIdentifiers" => [
0 => "us.zoom.xos"
]
"Attribution" => "Zoom"
"Program" => "/Library/PrivilegedHelperTools/us.zoom.ZoomDaemon"
"ProgramArguments" => [
0 => "/Library/PrivilegedHelperTools/us.zoom.ZoomDaemon"
]
"TeamIdentifier" => "BJ4HAAB9B3"
}
[...]
```

### Enumeracija

Moguće je **izlistati sve** konfigurisane pozadinske stavke pokretanjem Apple CLI alata:

```bash
# The tool will always ask for the users password
sfltool dumpbtm
```

Osim toga, takođe je moguće prikazati ovu informaciju pomoću [**DumpBTM**](https://github.com/objective-see/DumpBTM).

```bash
# You need to grant the Terminal Full Disk Access for this to work
chmod +x dumpBTM
xattr -rc dumpBTM # Remove quarantine attr
./dumpBTM
```

Ove informacije se čuvaju u **`/private/var/db/com.apple.backgroundtaskmanagement/BackgroundItems-v4.btm`** i Terminalu je potrebna FDA.

### Manipulacija sa BTM

Kada se pronađe nova upornost, događaj tipa **`ES_EVENT_TYPE_NOTIFY_BTM_LAUNCH_ITEM_ADD`** se javlja. Dakle, bilo koji način da se **spreči** slanje ovog **događaja** ili da se **agent obavesti** korisnik će pomoći napadaču da zaobiđe BTM.

* **Resetovanje baze podataka**: Pokretanje sledeće komande će resetovati bazu podataka (trebalo bi je ponovo izgraditi od početka), međutim, iz nekog razloga, nakon pokretanja ove komande, **nijedna nova upornost neće biti prijavljena sve dok se sistem ne ponovo pokrene**.
* Potreban je **root** pristup.

```bash
# Reset the database
sfltool resettbtm
```

* **Zaustavite Agenta**: Moguće je poslati signal zaustavljanja agentu kako se **ne bi obaveštavao korisnik** kada se pronađu nove detekcije.

```bash
# Get PID
pgrep BackgroundTaskManagementAgent
1011

# Stop it
kill -SIGSTOP 1011

# Check it's stopped (a T means it's stopped)
ps -o state 1011
T
```

* **Bag**: Ako **proces koji je stvorio upornost brzo završi**, demon će pokušati **dobiti informacije** o tome, **neće uspjeti**, i **neće moći poslati događaj** koji ukazuje da se nešto novo uporno.

Reference i **više informacija o BTM**:

* [https://youtu.be/9hjUmT031tc?t=26481](https://youtu.be/9hjUmT031tc?t=26481)
* [https://www.patreon.com/posts/new-developer-77420730?l=fr](https://www.patreon.com/posts/new-developer-77420730?l=fr)
* [https://support.apple.com/en-gb/guide/deployment/depdca572563/web](https://support.apple.com/en-gb/guide/deployment/depdca572563/web)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite vidjeti **oglašavanje vaše kompanije na HackTricks-u** ili **preuzeti HackTricks u PDF formatu**, provjerite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podijelite svoje hakirajuće trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
