# Windows Artifacts

## Windows Artifakti

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Generički Windows Artifakti

### Windows 10 Obaveštenja

Na putanji `\Users\<korisničko_ime>\AppData\Local\Microsoft\Windows\Notifications` možete pronaći bazu podataka `appdb.dat` (pre Windows Anniversary) ili `wpndatabase.db` (posle Windows Anniversary).

Unutar ove SQLite baze podataka, možete pronaći tabelu `Notification` sa svim obaveštenjima (u XML formatu) koja mogu sadržati interesantne podatke.

### Vremenska linija

Vremenska linija je karakteristika Windows-a koja pruža **hronološku istoriju** posećenih web stranica, izmenjenih dokumenata i izvršenih aplikacija.

Baza podataka se nalazi na putanji `\Users\<korisničko_ime>\AppData\Local\ConnectedDevicesPlatform\<id>\ActivitiesCache.db`. Ovu bazu podataka možete otvoriti sa alatom SQLite ili sa alatom [**WxTCmd**](https://github.com/EricZimmerman/WxTCmd) **koji generiše 2 fajla koji se mogu otvoriti sa alatom** [**TimeLine Explorer**](https://ericzimmerman.github.io/#!index.md).

### ADS (Alternate Data Streams)

Preuzeti fajlovi mogu sadržati **ADS Zone.Identifier** koji ukazuje **kako** je fajl **preuzet** sa intraneta, interneta, itd. Neki softveri (kao što su pretraživači) obično dodaju **još** **informacija** kao što je **URL** sa kog je fajl preuzet.

## **Rezervne kopije fajlova**

### Korpa za smeće

U Vista/Win7/Win8/Win10 operativnim sistemima, **Korpa za smeće** se može pronaći u folderu **`$Recycle.bin`** u korenu diska (`C:\$Recycle.bin`).\
Kada se fajl obriše u ovom folderu, kreiraju se 2 specifična fajla:

* `$I{id}`: Informacije o fajlu (datum kada je obrisan}
* `$R{id}`: Sadržaj fajla

![](<../../../.gitbook/assets/image (486).png>)

Koristeći ove fajlove, možete koristiti alat [**Rifiuti**](https://github.com/abelcheung/rifiuti2) da biste dobili originalnu adresu obrisanih fajlova i datum kada su obrisani (koristite `rifiuti-vista.exe` za Vista – Win10).

```
.\rifiuti-vista.exe C:\Users\student\Desktop\Recycle
```

![](<../../../.gitbook/assets/image (495) (1) (1) (1).png>)

### Kopije senki volumena

Shadow Copy je tehnologija koja je uključena u Microsoft Windows i može kreirati **rezervne kopije** ili snimke fajlova ili volumena računara, čak i kada su u upotrebi.

Ove rezervne kopije se obično nalaze u `\System Volume Information` od korena fajl sistema, a ime je sastavljeno od **UID-ova** prikazanih na sledećoj slici:

![](<../../../.gitbook/assets/image (520).png>)

Montiranjem forenzičke slike sa **ArsenalImageMounter**-om, alatka [**ShadowCopyView**](https://www.nirsoft.net/utils/shadow\_copy\_view.html) se može koristiti za pregledanje kopije senke i čak **izvlačenje fajlova** iz rezervnih kopija senke.

![](<../../../.gitbook/assets/image (521).png>)

Unos registra `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\BackupRestore` sadrži fajlove i ključeve **koji se neće rezervisati**:

![](<../../../.gitbook/assets/image (522).png>)

Registar `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\VSS` takođe sadrži informacije o konfiguraciji `Volume Shadow Copies`.

### Office automatski sačuvani fajlovi

Office automatski sačuvava fajlove na sledećoj lokaciji: `C:\Usuarios\\AppData\Roaming\Microsoft{Excel|Word|Powerpoint}\`

## Shell stavke

Shell stavka je stavka koja sadrži informacije o tome kako pristupiti drugom fajlu.

### Nedavni dokumenti (LNK)

Windows **automatski** **kreira** ove **prečice** kada korisnik **otvori, koristi ili kreira fajl** u:

* Win7-Win10: `C:\Users\\AppData\Roaming\Microsoft\Windows\Recent\`
* Office: `C:\Users\\AppData\Roaming\Microsoft\Office\Recent\`

Kada se kreira folder, takođe se kreira veza do foldera, roditeljskog foldera i pradedovskog foldera.

Ove automatski kreirane link fajlove **sadrže informacije o poreklu** kao da li je to **fajl** **ili** folder, **MAC** **vremena** tog fajla, **informacije o volumenu** gde je fajl smešten i **folder ciljnog fajla**. Ove informacije mogu biti korisne za oporavak tih fajlova u slučaju da su uklonjeni.

Takođe, **datum kreiranja linka** fajla je prvo **vreme** kada je originalni fajl **prvi put** **korišćen**, a **datum** **izmene** link fajla je **poslednje** **vreme** kada je origin fajl korišćen.

Za pregledanje ovih fajlova možete koristiti [**LinkParser**](http://4discovery.com/our-tools/).

U ovoj alatki ćete naći **2 seta** vremenskih oznaka:

* **Prvi set:**

1. FileModifiedDate
2. FileAccessDate
3. FileCreationDate

* **Drugi set:**

1. LinkModifiedDate
2. LinkAccessDate
3. LinkCreationDate.

Prvi set vremenskih oznaka se odnosi na **vremenske oznake samog fajla**. Drugi set se odnosi na **vremenske oznake povezanog fajla**.

Možete dobiti iste informacije pokretanjem Windows CLI alatke: [**LECmd.exe**](https://github.com/EricZimmerman/LECmd)

```
LECmd.exe -d C:\Users\student\Desktop\LNKs --csv C:\Users\student\Desktop\LNKs
```

U ovom slučaju, informacije će biti sačuvane unutar CSV datoteke.

### Jumpliste

Ovo su nedavne datoteke koje su označene po aplikacijama. To je lista **nedavnih datoteka koje je koristila aplikacija** kojoj možete pristupiti u svakoj aplikaciji. Mogu se **automatski kreirati ili biti prilagođene**.

Automatski kreirane **jumpliste** se čuvaju u `C:\Users\{korisničko_ime}\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\`. Jumpliste su nazvane prema formatu `{id}.autmaticDestinations-ms` gde je početni ID ID aplikacije.

Prilagođene jumpliste se čuvaju u `C:\Users\{korisničko_ime}\AppData\Roaming\Microsoft\Windows\Recent\CustomDestination\` i obično ih aplikacija kreira jer se nešto **važno** desilo sa datotekom (možda je označena kao omiljena).

Vreme kreiranja bilo koje jumpliste pokazuje **prvi put kada je datoteka pristupljena** i vreme izmene poslednji put.

Jumpliste možete pregledati koristeći [**JumplistExplorer**](https://ericzimmerman.github.io/#!index.md).

![](<../../../.gitbook/assets/image (474).png>)

(_Napomena: Vremenske oznake koje pruža JumplistExplorer odnose se na samu jumplist datoteku_)

### Shellbags

[**Pratite ovaj link da biste saznali šta su shellbags.**](interesting-windows-registry-keys.md#shellbags)

## Korišćenje Windows USB uređaja

Moguće je identifikovati da je USB uređaj korišćen zahvaljujući kreiranju:

* Windows Recent Folder
* Microsoft Office Recent Folder
* Jumpliste

Imajte na umu da neki LNK fajl umesto da pokazuje na originalnu putanju, pokazuje na WPDNSE folder:

![](<../../../.gitbook/assets/image (476).png>)

Datoteke u folderu WPDNSE su kopija originalnih datoteka, pa neće preživeti restart računara, a GUID se uzima iz shellbaga.

### Informacije iz registra

[Proverite ovu stranicu da biste saznali](interesting-windows-registry-keys.md#usb-information) koje registarske ključeve sadrže zanimljive informacije o povezanim USB uređajima.

### setupapi

Proverite datoteku `C:\Windows\inf\setupapi.dev.log` da biste dobili vremenske oznake kada je USB veza uspostavljena (pretražite `Section start`).

![](https://github.com/carlospolop/hacktricks/blob/rs/.gitbook/assets/image%20\(477\)%20\(2\)%20\(2\)%20\(2\)%20\(2\)%20\(2\)%20\(2\)%20\(2\)%20\(3\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(14\).png)

### USB Detective

[**USBDetective**](https://usbdetective.com) se može koristiti za dobijanje informacija o USB uređajima koji su bili povezani sa slikom.

![](<../../../.gitbook/assets/image (483).png>)

### Čišćenje Plug and Play

Zakazani zadatak poznat kao 'Plug and Play Cleanup' je pretežno dizajniran za uklanjanje zastarelih verzija drajvera. Suprotno od navedene svrhe zadržavanja najnovije verzije paketa drajvera, online izvori sugerišu da takođe cilja drajvere koji su bili neaktivni tokom 30 dana. Kao rezultat toga, drajveri za prenosive uređaje koji nisu bili povezani u poslednjih 30 dana mogu biti podložni brisanju.

Zadatak se nalazi na sledećoj putanji: `C:\Windows\System32\Tasks\Microsoft\Windows\Plug and Play\Plug and Play Cleanup`.

Prikazan je snimak ekrana sadržaja zadatka: ![](https://2.bp.blogspot.com/-wqYubtuR\_W8/W19bV5S9XyI/AAAAAAAANhU/OHsBDEvjqmg9ayzdNwJ4y2DKZnhCdwSMgCLcBGAs/s1600/xml.png)

**Ključni komponenti i podešavanja zadatka:**

* **pnpclean.dll**: Ova DLL je odgovorna za sam proces čišćenja.
* **UseUnifiedSchedulingEngine**: Postavljeno na `TRUE`, što ukazuje na korišćenje generičkog mehanizma zakazivanja zadataka.
* **MaintenanceSettings**:
* **Period ('P1M')**: Usmerava Task Scheduler da pokrene zadatak čišćenja mesečno tokom redovnog automatskog održavanja.
* **Deadline ('P2M')**: Nalaže Task Scheduleru, ako zadatak ne uspe dva uzastopna meseca, da izvrši zadatak tokom hitnog automatskog održavanja.

Ova konfiguracija obezbeđuje redovno održavanje i čišćenje drajvera, uz mogućnost ponovnog pokušaja izvršavanja zadatka u slučaju uzastopnih neuspeha.

**Za više informacija pogledajte:** [**https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html**](https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html)

## Emailovi

Emailovi sadrže **2 zanimljiva dela: zaglavlja i sadržaj** emaila. U **zaglavljima** možete pronaći informacije kao što su:

* **Ko** je poslao email (adresa e-pošte, IP adresa, poštanski serveri koji su preusmerili email)
* **Kada** je email poslat

Takođe, u zaglavljima `References` i `In-Reply-To` možete pronaći ID poruka:

![](<../../../.gitbook/assets/image (484).png>)

### Windows Mail aplikacija

Ova aplikacija čuva emailove u HTML ili tekstualnom formatu. Emailovi se mogu pronaći unutar podfoldera unutar `\Users\<korisničko_ime>\AppData\Local\Comms\Unistore\data\3\`. Emailovi se čuvaju sa ekstenzijom `.dat`.

**Metapodaci** emailova i **kontakti** mogu se pronaći unutar **EDB baze podataka**: `\Users\<korisničko_ime>\AppData\Local\Comms\UnistoreDB\store.vol`

**Promenite ekstenziju** datoteke iz `.vol` u `.edb` i možete koristiti alatku [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) da je otvorite. U tabeli `Message` možete videti emailove.

### Microsoft Outlook

Kada se koriste Exchange serveri ili Outlook klijenti, postojaće neka MAPI zaglavlja:

* `Mapi-Client-Submit-Time`: Vreme sistema kada je email poslat
* `Mapi-Conversation-Index`: Broj dečijih poruka u niti i vremenska oznaka svake poruke u niti
* `Mapi-Entry-ID`: Identifikator poruke.
* `Mappi-Message-Flags` i `Pr_last_Verb-Executed`: Informacije o MAPI klijentu (poruka pročitana? nepročitana? odgovorena? preusmerena? van kancelarije?)

U Microsoft Outlook klijentu, sve poslate/primljene poruke, podaci o kontaktima i podaci o kalendaru se čuvaju u PST datoteci na sledećoj putanji:

* `%USERPROFILE%\Local Settings\Application Data\Microsoft\Outlook` (WinXP)
* `%USERPROFILE%\AppData\Local\Microsoft\Outlook`

Putanja registra `HKEY_CURRENT_USER\Software\Microsoft\WindowsNT\CurrentVersion\Windows Messaging Subsystem\Profiles\Outlook` ukazuje na korišćenu datoteku.

PST datoteku možete otvoriti koristeći alatku [**Kernel PST Viewer**](https://www.nucleustechnologies.com/es/visor-de-pst.html).

![](<../../../.gitbook/assets/image (485).png>)

### Microsoft Outlook OST fajlovi

**OST fajl** se generiše od strane Microsoft Outlook-a kada je konfigurisan sa **IMAP** ili **Exchange** serverom, čuvajući slične informacije kao PST fajl. Ovaj fajl je sinhronizovan sa serverom i čuva podatke za **poslednjih 12 meseci** do **maksimalne veličine od 50GB**, i nalazi se u istom direktorijumu kao i PST fajl. Za pregled OST fajla, može se koristiti [**Kernel OST viewer**](https://www.nucleustechnologies.com/ost-viewer.html).

### Dobijanje priloga

Izgubljeni prilozi mogu biti povraćeni sa:

* Za **IE10**: `%APPDATA%\Local\Microsoft\Windows\Temporary Internet Files\Content.Outlook`
* Za **IE11 i novije**: `%APPDATA%\Local\Microsoft\InetCache\Content.Outlook`

### Thunderbird MBOX fajlovi

**Thunderbird** koristi **MBOX fajlove** za čuvanje podataka, smeštene na lokaciji `\Users\%USERNAME%\AppData\Roaming\Thunderbird\Profiles`.

### Sličice slika

* **Windows XP i 8-8.1**: Pregledanje foldera sa sličicama generiše `thumbs.db` fajl koji čuva prikaze slika, čak i nakon brisanja.
* **Windows 7/10**: `thumbs.db` se kreira prilikom pristupa preko mreže putem UNC putanje.
* **Windows Vista i novije**: Sličice slika su centralizovane u `%userprofile%\AppData\Local\Microsoft\Windows\Explorer` sa fajlovima nazvanim **thumbcache\_xxx.db**. Alati [**Thumbsviewer**](https://thumbsviewer.github.io) i [**ThumbCache Viewer**](https://thumbcacheviewer.github.io) se koriste za pregledanje ovih fajlova.

### Informacije iz Windows registra

Windows registar, koji čuva obimne podatke o aktivnostima sistema i korisnika, nalazi se u fajlovima:

* `%windir%\System32\Config` za različite `HKEY_LOCAL_MACHINE` podključeve.
* `%UserProfile%{User}\NTUSER.DAT` za `HKEY_CURRENT_USER`.
* Windows Vista i novije verzije čuvaju rezervne kopije `HKEY_LOCAL_MACHINE` registarskih fajlova u `%Windir%\System32\Config\RegBack\`.
* Dodatno, informacije o izvršavanju programa se čuvaju u `%UserProfile%\{User}\AppData\Local\Microsoft\Windows\USERCLASS.DAT` od Windows Vista i Windows 2008 Server verzija nadalje.

### Alati

Neki alati su korisni za analizu registarskih fajlova:

* **Registry Editor**: Instaliran je u Windows-u. To je grafički interfejs za navigaciju kroz Windows registar trenutne sesije.
* [**Registry Explorer**](https://ericzimmerman.github.io/#!index.md): Omogućava učitavanje registarskog fajla i navigaciju kroz njega pomoću grafičkog interfejsa. Takođe sadrži obeleživače koji ističu ključeve sa interesantnim informacijama.
* [**RegRipper**](https://github.com/keydet89/RegRipper3.0): Ima grafički interfejs koji omogućava navigaciju kroz učitani registar i takođe sadrži dodatke koji ističu interesantne informacije unutar učitanog registra.
* [**Windows Registry Recovery**](https://www.mitec.cz/wrr.html): Još jedna aplikacija sa grafičkim interfejsom koja je sposobna da izvuče važne informacije iz učitanog registra.

### Povraćaj obrisanih elemenata

Kada se ključ obriše, označava se kao takav, ali se neće ukloniti sve dok prostor koji zauzima ne bude potreban. Stoga, korišćenjem alata kao što je **Registry Explorer** moguće je povratiti ove obrisane ključeve.

### Vreme poslednje izmene

Svaki ključ-vrednost sadrži **vremensku oznaku** koja pokazuje kada je poslednji put izmenjen.

### SAM

Fajl/hive **SAM** sadrži heševe **korisnika, grupa i lozinki korisnika** sistema.

U `SAM\Domains\Account\Users` možete dobiti korisničko ime, RID, poslednju prijavu, poslednji neuspeli pokušaj prijave, brojač prijava, politiku lozinke i kada je nalog kreiran. Da biste dobili **heševe**, takođe **trebate** fajl/hive **SYSTEM**.

### Interesantni unosi u Windows registru

{% content-ref url="interesting-windows-registry-keys.md" %}
[interesting-windows-registry-keys.md](interesting-windows-registry-keys.md)
{% endcontent-ref %}

## Izvršeni programi

### Osnovni Windows procesi

U [ovom postu](https://jonahacks.medium.com/investigating-common-windows-processes-18dee5f97c1d) možete saznati o uobičajenim Windows procesima kako biste otkrili sumnjive ponašanja.

### Nedavni Windows programi

Unutar registra `NTUSER.DAT` na putanji `Software\Microsoft\Current Version\Search\RecentApps` možete pronaći podključeve sa informacijama o **izvršenim aplikacijama**, **poslednjem vremenu** izvršavanja i **broju puta** koliko su pokrenute.

### BAM (Background Activity Moderator)

Možete otvoriti fajl `SYSTEM` sa registarskim editorom i unutar putanje `SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}` možete pronaći informacije o **aplikacijama izvršenim od strane svakog korisnika** (obratite pažnju na `{SID}` u putanji) i **vremenu** kada su izvršene (vreme se nalazi unutar vrednosti podataka registra).

### Windows Prefetch

Prefetching je tehnika koja omogućava računaru da tiho **preuzme neophodne resurse potrebne za prikaz sadržaja** kojem korisnik **može pristupiti u bliskoj budućnosti**, kako bi se resursi mogli brže pristupiti.

Windows prefetch se sastoji od kreiranja **keševa izvršenih programa** kako bi se mogli brže učitati. Ovi keševi se kreiraju kao `.pf` fajlovi unutar putanje: `C:\Windows\Prefetch`. Postoji ograničenje od 128 fajlova u XP/VISTA/WIN7 i 1024 fajla u Win8/Win10.

Naziv fajla se kreira kao `{ime_programa}-{hash}.pf` (hash se bazira na putanji i argumentima izvršnog fajla). U W10 su ovi fajlovi kompresovani. Imajte na umu da samo prisustvo fajla ukazuje da je **program izvršen** u nekom trenutku.

Fajl `C:\Windows\Prefetch\Layout.ini` sadrži **nazive foldera fajlova koji su prefetch-ovani**. Ovaj fajl sadrži **informacije o broju izvršavanja**, **datumima** izvršavanja i **fajlovima** **otvorenim** od strane programa.

Za pregledanje ovih fajlova možete koristiti alat [**PEcmd.exe**](https://github.com/EricZimmerman/PECmd):

```bash
.\PECmd.exe -d C:\Users\student\Desktop\Prefetch --html "C:\Users\student\Desktop\out_folder"
```

![](<../../../.gitbook/assets/image (487).png>)

### Superprefetch

**Superprefetch** ima isti cilj kao i prefetch, **brže učitavanje programa** predviđanjem šta će se sledeće učitati. Međutim, ne zamenjuje prefetch servis.\
Ovaj servis generiše bazu podataka u `C:\Windows\Prefetch\Ag*.db`.

U ovim bazama podataka možete pronaći **ime** **programa**, **broj** **izvršavanja**, **otvorene** **datoteke**, **pristupane** **particije**, **kompletan** **putanja**, **vremenski okviri** i **vremenske oznake**.

Ove informacije možete pristupiti pomoću alata [**CrowdResponse**](https://www.crowdstrike.com/resources/community-tools/crowdresponse/).

### SRUM

**System Resource Usage Monitor** (SRUM) **prati** **resurse** **koje proces koristi**. Pojavio se u W8 i podatke čuva u ESE bazi podataka smeštenoj u `C:\Windows\System32\sru\SRUDB.dat`.

Daje sledeće informacije:

* AppID i putanja
* Korisnik koji je izvršio proces
* Poslati bajtovi
* Primljeni bajtovi
* Mrežni interfejs
* Trajanje veze
* Trajanje procesa

Ove informacije se ažuriraju svakih 60 minuta.

Možete dobiti podatke iz ovog fajla koristeći alat [**srum\_dump**](https://github.com/MarkBaggett/srum-dump).

```bash
.\srum_dump.exe -i C:\Users\student\Desktop\SRUDB.dat -t SRUM_TEMPLATE.xlsx -o C:\Users\student\Desktop\srum
```

### AppCompatCache (ShimCache)

**AppCompatCache**, poznat i kao **ShimCache**, čini deo **Baze podataka o kompatibilnosti aplikacija** koju je razvio **Microsoft** kako bi rešio probleme sa kompatibilnošću aplikacija. Ovaj sistemski komponent beleži različite metapodatke datoteka, koji uključuju:

* Puni put do datoteke
* Veličinu datoteke
* Vreme poslednje izmene pod **$Standard\_Information** (SI)
* Vreme poslednjeg ažuriranja ShimCache-a
* Zastavicu izvršenja procesa

Takvi podaci se čuvaju u registru na određenim lokacijama, u zavisnosti od verzije operativnog sistema:

* Za XP, podaci se čuvaju pod `SYSTEM\CurrentControlSet\Control\SessionManager\Appcompatibility\AppcompatCache` sa kapacitetom od 96 unosa.
* Za Server 2003, kao i za verzije Windowsa 2008, 2012, 2016, 7, 8 i 10, putanja za čuvanje je `SYSTEM\CurrentControlSet\Control\SessionManager\AppcompatCache\AppCompatCache`, sa kapacitetom od 512, odnosno 1024 unosa.

Za analizu čuvanih informacija preporučuje se korišćenje alata [**AppCompatCacheParser**](https://github.com/EricZimmerman/AppCompatCacheParser).

![](<../../../.gitbook/assets/image (488).png>)

### Amcache

Datoteka **Amcache.hve** je suštinski registarski hive koji beleži detalje o aplikacijama koje su izvršene na sistemu. Obično se nalazi na putanji `C:\Windows\AppCompat\Programas\Amcache.hve`.

Ova datoteka je značajna jer čuva zapise o nedavno izvršenim procesima, uključujući putanje do izvršnih datoteka i njihove SHA1 heš vrednosti. Ove informacije su neprocenjive za praćenje aktivnosti aplikacija na sistemu.

Za izdvajanje i analizu podataka iz **Amcache.hve** datoteke može se koristiti alat [**AmcacheParser**](https://github.com/EricZimmerman/AmcacheParser). Sledeća komanda je primer kako koristiti AmcacheParser za analizu sadržaja datoteke **Amcache.hve** i izlaz rezultata u CSV formatu:

```bash
AmcacheParser.exe -f C:\Users\genericUser\Desktop\Amcache.hve --csv C:\Users\genericUser\Desktop\outputFolder
```

Među generisanim CSV datotekama, posebno je značajna datoteka `Amcache_Unassociated file entries` zbog bogatih informacija koje pruža o nepovezanim unosima datoteka.

Najinteresantnija generisana CSV datoteka je `Amcache_Unassociated file entries`.

### RecentFileCache

Ovaj artefakt se može pronaći samo u W7 u `C:\Windows\AppCompat\Programs\RecentFileCache.bcf` i sadrži informacije o nedavnom izvršavanju određenih binarnih datoteka.

Možete koristiti alat [**RecentFileCacheParse**](https://github.com/EricZimmerman/RecentFileCacheParser) za parsiranje datoteke.

### Zakazani zadaci

Možete ih izvući iz `C:\Windows\Tasks` ili `C:\Windows\System32\Tasks` i čitati ih kao XML.

### Servisi

Možete ih pronaći u registru pod `SYSTEM\ControlSet001\Services`. Možete videti šta će biti izvršeno i kada.

### **Windows Store**

Instalirane aplikacije mogu se pronaći u `\ProgramData\Microsoft\Windows\AppRepository\`\
Ovaj repozitorijum ima **log** sa **svakom instaliranom aplikacijom** u sistemu unutar baze podataka **`StateRepository-Machine.srd`**.

Unutar tabele Application ove baze podataka, mogu se pronaći kolone: "Application ID", "PackageNumber" i "Display Name". Ove kolone sadrže informacije o preinstaliranim i instaliranim aplikacijama, a može se utvrditi da li su neke aplikacije deinstalirane jer bi ID-jevi instaliranih aplikacija trebali biti uzastopni.

Takođe je moguće **pronaći instalirane aplikacije** unutar putanje registra: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Applications\`\
I **deinstalirane aplikacije** u: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Deleted\`

## Windows događaji

Informacije koje se pojavljuju unutar Windows događaja su:

* Šta se desilo
* Vremenska oznaka (UTC + 0)
* Uključeni korisnici
* Uključeni hostovi (ime računara, IP adresa)
* Pristupani resursi (datoteke, folderi, štampači, servisi)

Logovi se nalaze u `C:\Windows\System32\config` pre Windows Viste i u `C:\Windows\System32\winevt\Logs` posle Windows Viste. Pre Windows Viste, logovi događaja su bili u binarnom formatu, a posle toga su u **XML formatu** i koriste **.evtx** ekstenziju.

Lokacija datoteka događaja može se pronaći u registru sistema u **`HKLM\SYSTEM\CurrentControlSet\services\EventLog\{Application|System|Security}`**

Mogu se vizualizovati putem Windows Event Viewer-a (**`eventvwr.msc`**) ili drugim alatima poput [**Event Log Explorer**](https://eventlogxp.com) **ili** [**Evtx Explorer/EvtxECmd**](https://ericzimmerman.github.io/#!index.md)**.**

## Razumevanje beleženja događaja o bezbednosti u Windows-u

Pristupni događaji se beleže u konfiguracionoj datoteci bezbednosti koja se nalazi na lokaciji `C:\Windows\System32\winevt\Security.evtx`. Veličina ove datoteke je podesiva, a kada se dostigne kapacitet, stariji događaji se prepisuju. Beleženi događaji uključuju prijavljivanje i odjavljivanje korisnika, korisničke radnje i promene u postavkama bezbednosti, kao i pristupanje datotekama, folderima i deljenim resursima.

### Ključni ID-jevi događaja za autentifikaciju korisnika:

* **EventID 4624**: Ukazuje na uspešnu autentifikaciju korisnika.
* **EventID 4625**: Označava neuspešnu autentifikaciju.
* **EventID 4634/4647**: Predstavljaju događaje odjavljivanja korisnika.
* **EventID 4672**: Označava prijavljivanje sa administratorskim privilegijama.

#### Podtipovi unutar EventID 4634/4647:

* **Interactive (2)**: Direktno prijavljivanje korisnika.
* **Network (3)**: Pristup deljenim fasciklama.
* **Batch (4)**: Izvršavanje batch procesa.
* **Service (5)**: Pokretanje servisa.
* **Proxy (6)**: Proksi autentifikacija.
* **Unlock (7)**: Otključavanje ekrana lozinkom.
* **Network Cleartext (8)**: Prenos lozinke u čistom tekstu, često od strane IIS-a.
* **New Credentials (9)**: Korišćenje drugih akreditiva za pristup.
* **Remote Interactive (10)**: Prijavljivanje putem udaljenog radnog okruženja ili terminalnih usluga.
* **Cache Interactive (11)**: Prijavljivanje sa keširanim akreditivima bez kontakta sa kontrolerom domena.
* **Cache Remote Interactive (12)**: Udaljeno prijavljivanje sa keširanim akreditivima.
* **Cached Unlock (13)**: Otključavanje sa keširanim akreditivima.

#### Statusni i podstatusni kodovi za EventID 4625:

* **0xC0000064**: Korisničko ime ne postoji - Može ukazivati na napad enumeracije korisničkih imena.
* **0xC000006A**: Ispravno korisničko ime, ali pogrešna lozinka - Mogući pokušaj nagađanja ili napad metodom isprobavanja svih mogućih kombinacija lozinki.
* **0xC0000234**: Korisnički nalog zaključan - Može pratiti napad metodom isprobavanja svih mogućih kombinacija lozinki koji rezultira višestrukim neuspelim prijavljivanjima.
* **0xC0000072**: Onemogućen nalog - Neovlašćeni pokušaji pristupa onemogućenim nalozima.
* **0xC000006F**: Prijavljivanje van dozvoljenog vremena - Ukazuje na pokušaje pristupa van postavljenih vremenskih okvira za prijavljivanje, što može biti znak neovlašćenog pristupa.
* **0xC0000070**: Kršenje ograničenja radne stanice - Može biti pokušaj prijavljivanja sa neovlašćene lokacije.
* **0xC0000193**: Isteče vreme naloga - Pokušaji pristupa sa isteklim korisničkim nalozima.
* **0xC0000071**: Istečena lozinka - Pokušaji prijavljivanja sa zastarelim lozinkama.
* **0xC0000133**: Problemi sa sinhronizacijom vremena - Velike razlike u vremenu između klijenta i servera mogu ukazivati na sofisticiranije napade poput "pass-the-ticket".
* **0xC0000224**: Obavezna promena lozinke - Česte obavezne promene mogu ukazivati na pokušaj narušavanja sigurnosti naloga.
* **0xC0000225**: Ukazuje na grešku u sistemu, a ne na sigurnosni problem.
* **0xC000015b**: Odbijen tip prijavljivanja - Pokušaj pristupa sa neovlašćenim tipom prijavljivanja, kao što je pokušaj korisnika da izvrši prijavljivanje servisa.

#### EventID 4616:

* **Promena vremena**: Izmena sistemskog vremena, može otežati forenzičku analizu događaja.

#### EventID 6005 i 6006:

* **Pokretanje i gašenje sistema**: EventID 6005 označava pokretanje sistema, dok EventID 6006 označava gašenje sistema.

#### EventID 1102:

* **Brisanje logova**: Brisanje sigurnosnih logova, što često ukazuje na pokušaj prikrivanja nezakonitih aktivnosti.

#### EventID-ovi za praćenje USB uređaja:

* **20001 / 20003 / 10000**: Prvo povezivanje USB uređaja.
* **10100**: Ažuriranje drajvera USB uređaja.
* **EventID 112**: Vreme umetanja USB uređaja.

Za praktične primere simuliranja ovih vrsta prijavljivanja i prilika za izvlačenje akreditiva, pogledajte detaljan vodič \[Altered Security]\(https://www.alteredsecurity

#### Događaji o napajanju sistema

EventID 6005 označava pokretanje sistema, dok EventID 6006 označava gašenje.

#### Brisanje logova

Security EventID 1102 signalizira brisanje logova, što je kritičan događaj za forenzičku analizu.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
