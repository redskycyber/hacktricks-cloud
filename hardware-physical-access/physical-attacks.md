# Physical Attacks

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Obnova BIOS lozinke i sistemsko obezbeđenje

**Resetovanje BIOS-a** može se postići na nekoliko načina. Većina matičnih ploča uključuje **bateriju** koja, kada se ukloni oko **30 minuta**, će resetovati BIOS podešavanja, uključujući i lozinku. Alternativno, **jumper na matičnoj ploči** može se podesiti da resetuje ova podešavanja povezivanjem određenih pinova.

U situacijama kada hardverske izmene nisu moguće ili praktične, **softverski alati** nude rešenje. Pokretanje sistema sa **Live CD/USB** distribucijama kao što je **Kali Linux** omogućava pristup alatima poput _**killCmos**_ i _**CmosPWD**_, koji mogu pomoći u obnovi BIOS lozinke.

U slučajevima kada je BIOS lozinka nepoznata, unošenje lozinke netačno **tri puta** obično rezultira greškom. Ovaj kod se može koristiti na veb sajtovima poput [https://bios-pw.org](https://bios-pw.org) kako bi se potencijalno dobio upotrebljiva lozinka.

### UEFI bezbednost

Za moderne sisteme koji koriste **UEFI** umesto tradicionalnog BIOS-a, alat **chipsec** se može koristiti za analizu i modifikaciju UEFI podešavanja, uključujući onemogućavanje **Secure Boot**-a. To se može postići sledećom komandom:

`python chipsec_main.py -module exploits.secure.boot.pk`

### Analiza RAM-a i napadi sa hladnim startom

RAM zadržava podatke kratko vreme nakon isključivanja napajanja, obično **1 do 2 minuta**. Ova postojanost se može produžiti na **10 minuta** primenom hladnih supstanci, poput tečnog azota. Tokom ovog produženog perioda, može se napraviti **damp memorije** korišćenjem alata poput **dd.exe** i **volatility** za analizu.

### Napadi sa direktnim pristupom memoriji (DMA)

**INCEPTION** je alat dizajniran za **fizičku manipulaciju memorijom** putem DMA, kompatibilan sa interfejsima poput **FireWire**-a i **Thunderbolt**-a. Omogućava zaobilaženje postupka prijavljivanja tako što menja memoriju da prihvati bilo koju lozinku. Međutim, neefikasan je protiv sistema sa **Windows 10**.

### Live CD/USB za pristup sistemu

Promena sistemskih binarnih fajlova poput _**sethc.exe**_ ili _**Utilman.exe**_ sa kopijom _**cmd.exe**_ može obezbediti komandnu liniju sa sistemskim privilegijama. Alati poput **chntpw** mogu se koristiti za uređivanje **SAM** fajla Windows instalacije, što omogućava promenu lozinki.

**Kon-Boot** je alat koji olakšava prijavljivanje na Windows sisteme bez poznavanja lozinke privremeno modifikujući Windows kernel ili UEFI. Više informacija možete pronaći na [https://www.raymond.cc](https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/).

### Rad sa Windows bezbednosnim funkcijama

#### Prečice za pokretanje i oporavak

* **Supr**: Pristup BIOS podešavanjima.
* **F8**: Ulazak u režim oporavka.
* Pritisak na **Shift** nakon Windows banera može zaobići automatsko prijavljivanje.

#### Loši USB uređaji (BAD USB)

Uređaji poput **Rubber Ducky**-ja i **Teensyduino**-a služe kao platforme za kreiranje **loših USB** uređaja, sposobnih za izvršavanje unapred definisanih payload-a kada su povezani sa ciljnim računarom.

#### Kopiranje senzitivnih fajlova pomoću Volume Shadow Copy

Administrator privilegije omogućavaju kreiranje kopija osetljivih fajlova, uključujući **SAM** fajl, putem PowerShell-a.

### Zaobilaženje BitLocker enkripcije

BitLocker enkripcija može potencijalno biti zaobiđena ako se **recovery password** pronađe unutar fajla sa dump-om memorije (**MEMORY.DMP**). Alati poput **Elcomsoft Forensic Disk Decryptor**-a ili **Passware Kit Forensic**-a mogu se koristiti u tu svrhu.

### Društveno inženjering za dodavanje ključa za oporavak

Novi BitLocker ključ za oporavak može se dodati putem taktika društvenog inženjeringa, ubedljivanjem korisnika da izvrši komandu koja dodaje novi ključ za oporavak sastavljen od nula, čime se pojednostavljuje proces dešifrovanja.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
