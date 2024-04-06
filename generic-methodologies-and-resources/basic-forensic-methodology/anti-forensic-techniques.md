# Anti-Forensic Techniques

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Vremenske oznake

Napadač može biti zainteresovan za **menjanje vremenskih oznaka datoteka** kako bi izbegao otkrivanje.\
Moguće je pronaći vremenske oznake unutar MFT-a u atributima `$STANDARD_INFORMATION` \_\_ i \_\_ `$FILE_NAME`.

Oba atributa imaju 4 vremenske oznake: **Modifikacija**, **pristup**, **kreiranje** i **modifikacija MFT registra** (MACE ili MACB).

**Windows explorer** i druge alatke prikazuju informacije iz **`$STANDARD_INFORMATION`**.

### TimeStomp - Anti-forenzički alat

Ovaj alat **menja** informacije o vremenskim oznakama unutar **`$STANDARD_INFORMATION`** **ali ne** i informacije unutar **`$FILE_NAME`**. Zbog toga je moguće **identifikovati** **sumnjive** **aktivnosti**.

### Usnjrnl

**USN Journal** (Update Sequence Number Journal) je funkcija NTFS (Windows NT fajl sistem) koja prati promene na volumenu. Alatka [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) omogućava pregled ovih promena.

![](<../../.gitbook/assets/image (449).png>)

Prethodna slika je **izlaz** prikazan od strane **alatke** gde se može primetiti da su neke **promene izvršene** na datoteci.

### $LogFile

**Sve promene metapodataka na fajl sistemu se beleže** u procesu poznatom kao [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead\_logging). Beleženi metapodaci se čuvaju u fajlu nazvanom `**$LogFile**`, smeštenom u korenom direktorijumu NTFS fajl sistema. Alatke poput [LogFileParser](https://github.com/jschicht/LogFileParser) se mogu koristiti za parsiranje ovog fajla i identifikaciju promena.

![](<../../.gitbook/assets/image (450).png>)

Ponovo, u izlazu alatke je moguće videti da su **izvršene neke promene**.

Korišćenjem iste alatke moguće je identifikovati **kada su vremenske oznake modifikovane**:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: Vreme kreiranja fajla
* ATIME: Vreme modifikacije fajla
* MTIME: Vreme modifikacije MFT registra fajla
* RTIME: Vreme pristupa fajlu

### Poređenje `$STANDARD_INFORMATION` i `$FILE_NAME`

Još jedan način za identifikaciju sumnjivo modifikovanih datoteka je poređenje vremena na oba atributa u potrazi za **neslaganjima**.

### Nanosekunde

Vremenske oznake **NTFS** imaju **preciznost** od **100 nanosekundi**. Zato je veoma sumnjivo pronaći datoteke sa vremenskim oznakama poput 2010-10-10 10:10:**00.000:0000**.

### SetMace - Anti-forenzički alat

Ovaj alat može modifikovati oba atributa `$STARNDAR_INFORMATION` i `$FILE_NAME`. Međutim, od Windows Viste, potrebno je da operativni sistem bude uključen kako bi se ove informacije modifikovale.

## Sakrivanje podataka

NFTS koristi klaster i minimalnu veličinu informacija. To znači da ako datoteka zauzima klaster i po, **preostali pola klastera nikada neće biti korišćen** sve dok datoteka ne bude obrisana. Zato je moguće **sakriti podatke u ovom prostoru**.

Postoje alatke poput slacker koje omogućavaju sakrivanje podataka u ovom "skrivenom" prostoru. Međutim, analiza `$logfile` i `$usnjrnl` može pokazati da su neki podaci dodati:

![](<../../.gitbook/assets/image (452).png>)

Zatim, moguće je povratiti prostor korišćenjem alatki poput FTK Imager. Imajte na umu da ovakve alatke mogu sačuvati sadržaj obfuskovan ili čak šifrovan.

## UsbKill

Ovo je alatka koja će **isključiti računar ako se detektuje bilo kakva promena na USB** portovima.\
Način da se ovo otkrije je da se pregledaju pokrenuti procesi i **pregledaju svi pokrenuti Python skriptovi**.

## Linux distribucije uživo

Ove distribucije se **izvršavaju unutar RAM** memorije. Jedini način da se otkriju je **ako je NTFS fajl-sistem montiran sa dozvolama za pisanje**. Ako je montiran samo sa dozvolama za čitanje, neće biti moguće otkriti upad.

## Sigurno brisanje

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

## Windows konfiguracija

Moguće je onemogućiti nekoliko metoda beleženja u Windows-u kako bi se forenzička istraga otežala.

### Onemogućavanje vremenskih oznaka - UserAssist

Ovo je registarski ključ koji čuva datume i sate kada je svaki izvršni fajl pokrenut od strane korisnika.

Onemogućavanje UserAssist zahteva dva koraka:

1. Postavite dva registarska ključa, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` i `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, oba na nulu kako biste signalizirali da želite da se UserAssist onemogući.
2. Obrišite podstabla registra koja izgledaju kao `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>`.

### Onemogućavanje vremenskih oznaka - Prefetch

Ovo će sačuvati informacije o aplikacijama koje su pokrenute u cilju poboljšanja performansi Windows sistema. Međutim, ovo takođe može biti korisno za forenzičke prakse.

* Izvršite `regedit`
* Izaberite putanju fajla `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Desnim klikom na `EnablePrefetcher` i `EnableSuperfetch`
* Izaberite Modify na svakom od njih da biste promenili vrednost sa 1 (ili 3) na 0
* Restartujte

### Onemogućavanje vremenskih oznaka - Vreme poslednjeg pristupa

Svaki put kada se otvori folder sa NTFS volumena na Windows NT serveru, sistem uzima vreme da **ažurira vremensko polje na svakom navedenom folderu**, nazvano vreme poslednjeg

### Brisanje istorije USB uređaja

Svi unosi o **USB uređajima** se čuvaju u Windows registru pod ključem **USBSTOR** koji sadrži podključeve koji se kreiraju svaki put kada priključite USB uređaj na računar. Ovaj ključ se može pronaći ovde: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Brisanjem ovog ključa** ćete obrisati istoriju USB uređaja.\
Takođe možete koristiti alatku [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) da biste bili sigurni da ste ih obrisali (i da biste ih obrisali).

Još jedan fajl koji čuva informacije o USB uređajima je fajl `setupapi.dev.log` unutar `C:\Windows\INF`. Ovaj fajl takođe treba obrisati.

### Onemogući Shadow kopije

**Izlistajte** shadow kopije sa `vssadmin list shadowstorage`\
**Obrišite** ih pokretanjem `vssadmin delete shadow`

Takođe ih možete obrisati putem grafičkog interfejsa prateći korake predložene na [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Da biste onemogućili shadow kopije, sledite korake sa [ovog linka](https://support.waters.com/KB\_Inf/Other/WKB15560\_How\_to\_disable\_Volume\_Shadow\_Copy\_Service\_VSS\_in\_Windows):

1. Otvorite program Services tako što ćete u tekstualnom pretraživaču kucati "services" nakon što kliknete na dugme za pokretanje Windowsa.
2. Na listi pronađite "Volume Shadow Copy", izaberite ga, a zatim pristupite Properties opciji desnim klikom.
3. Izaberite Disabled iz padajućeg menija "Startup type", a zatim potvrdite promenu klikom na Apply i OK.

Takođe je moguće izmeniti konfiguraciju kojih fajlova će biti kopirano u shadow kopiju u registru `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

### Prepisivanje obrisanih fajlova

* Možete koristiti **Windows alatku**: `cipher /w:C` Ovo će narediti cipher-u da ukloni sve podatke sa dostupnog neiskorišćenog prostora na disku C.
* Takođe možete koristiti alatke kao što je [**Eraser**](https://eraser.heidi.ie)

### Brisanje Windows događajnih logova

* Windows + R --> eventvwr.msc --> Proširite "Windows Logs" --> Desni klik na svaku kategoriju i izaberite "Clear Log"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

### Onemogući Windows događajne logove

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* Unutar sekcije Services onemogućite servis "Windows Event Log"
* `WEvtUtil.exec clear-log` ili `WEvtUtil.exe cl`

### Onemogući $UsnJrnl

* `fsutil usn deletejournal /d c:`

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje tako što ćete slati PR-ove na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
