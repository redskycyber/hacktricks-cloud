# Checklist - Local Windows Privilege Escalation

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

### **Najbolji alat za traženje vektora lokalnog eskaliranja privilegija u Windows-u:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

### [Informacije o sistemu](windows-local-privilege-escalation/#system-info)

* [ ] Dobiti [**Informacije o sistemu**](windows-local-privilege-escalation/#system-info)
* [ ] Pretražiti **kernel** [**eksploate korišćenjem skripti**](windows-local-privilege-escalation/#version-exploits)
* [ ] Koristiti **Google za pretragu** kernel **eksploatacija**
* [ ] Koristiti **searchsploit za pretragu** kernel **eksploatacija**
* [ ] Zanimljive informacije u [**env varijablama**](windows-local-privilege-escalation/#environment)?
* [ ] Lozinke u [**PowerShell istoriji**](windows-local-privilege-escalation/#powershell-history)?
* [ ] Zanimljive informacije u [**Internet postavkama**](windows-local-privilege-escalation/#internet-settings)?
* [ ] [**Diskovi**](windows-local-privilege-escalation/#drives)?
* [ ] [**WSUS eksploatacija**](windows-local-privilege-escalation/#wsus)?
* [ ] [**AlwaysInstallElevated**](windows-local-privilege-escalation/#alwaysinstallelevated)?

### [Enumeracija logovanja/AV-a](windows-local-privilege-escalation/#enumeration)

* [ ] Proveriti [**Audit** ](windows-local-privilege-escalation/#audit-settings)i [**WEF** ](windows-local-privilege-escalation/#wef)postavke
* [ ] Proveriti [**LAPS**](windows-local-privilege-escalation/#laps)
* [ ] Proveriti da li je [**WDigest** ](windows-local-privilege-escalation/#wdigest)aktivan
* [ ] [**LSA Zaštita**](windows-local-privilege-escalation/#lsa-protection)?
* [ ] [**Zaštita Kredencijala**](windows-local-privilege-escalation/#credentials-guard)[?](windows-local-privilege-escalation/#cached-credentials)
* [ ] [**Keširane Kredencijale**](windows-local-privilege-escalation/#cached-credentials)?
* [ ] Proveriti da li postoji neki [**AV**](https://github.com/carlospolop/hacktricks/blob/rs/windows-hardening/windows-av-bypass/README.md)
* [ ] [**AppLocker Politika**](https://github.com/carlospolop/hacktricks/blob/rs/windows-hardening/authentication-credentials-uac-and-efs/README.md#applocker-policy)?
* [ ] [**UAC**](https://github.com/carlospolop/hacktricks/blob/rs/windows-hardening/authentication-credentials-uac-and-efs/uac-user-account-control/README.md)
* [ ] [**Korisničke Privilegije**](windows-local-privilege-escalation/#users-and-groups)
* [ ] Proveriti [**trenutne** korisničke **privilegije**](windows-local-privilege-escalation/#users-and-groups)
* [ ] Da li ste [**član neke privilegovane grupe**](windows-local-privilege-escalation/#privileged-groups)?
* [ ] Proveriti da li su vam omogućeni [bilo koji od ovih tokena](windows-local-privilege-escalation/#token-manipulation): **SeImpersonatePrivilege, SeAssignPrimaryPrivilege, SeTcbPrivilege, SeBackupPrivilege, SeRestorePrivilege, SeCreateTokenPrivilege, SeLoadDriverPrivilege, SeTakeOwnershipPrivilege, SeDebugPrivilege** ?
* [ ] [**Korisničke Sesije**](windows-local-privilege-escalation/#logged-users-sessions)?
* [ ] Proveriti [**korisničke direktorijume**](windows-local-privilege-escalation/#home-folders) (pristup?)
* [ ] Proveriti [**Politiku Lozinke**](windows-local-privilege-escalation/#password-policy)
* [ ] Šta je [**unutar Clipboard-a**](windows-local-privilege-escalation/#get-the-content-of-the-clipboard)?

### [Mreža](windows-local-privilege-escalation/#network)

* Proveriti **trenutne** [**informacije o mreži**](windows-local-privilege-escalation/#network)
* Proveriti **skrivene lokalne servise** ograničene prema spolja

### [Pokrenuti Procesi](windows-local-privilege-escalation/#running-processes)

* Dozvole fajlova i direktorijuma za procese binarne [**datoteke**](windows-local-privilege-escalation/#file-and-folder-permissions)
* [**Izvlačenje lozinki iz memorije**](windows-local-privilege-escalation/#memory-password-mining)
* [**Nesigurne GUI aplikacije**](windows-local-privilege-escalation/#insecure-gui-apps)
* Ukrasti kredencijale sa **zanimljivih procesa** putem `ProcDump.exe` ? (firefox, chrome, itd ...)

### [Servisi](windows-local-privilege-escalation/#services)

* [Možete li **modifikovati neki servis**?](windows-local-privilege-escalation/#permissions)
* [Možete li **modifikovati** binarni **fajl** koji se **izvršava** od strane bilo kog **servisa**?](windows-local-privilege-escalation/#modify-service-binary-path)
* [Možete li **modifikovati** registar bilo kog **servisa**?](windows-local-privilege-escalation/#services-registry-modify-permissions)
* [Možete li iskoristiti bilo koji **servis sa neispravnim putem binarnog fajla**?](windows-local-privilege-escalation/#unquoted-service-paths)

### [**Aplikacije**](windows-local-privilege-escalation/#applications)

* **Dozvole za pisanje na instalirane aplikacije**]\(windows-local-privilege-escalation/#write-permissions)
* [**Aplikacije koje se pokreću pri pokretanju**](windows-local-privilege-escalation/#run-at-startup)
* **Ranjivi** [**Driveri**](windows-local-privilege-escalation/#drivers)

### [DLL Hijacking](windows-local-privilege-escalation/#path-dll-hijacking)

* [ ] Možete li **pisati u bilo kojem folderu unutar PATH-a**?
* [ ] Postoji li poznati servisni binarni fajl koji **pokušava učitati nepostojeći DLL**?
* [ ] Možete li **pisati** u bilo kojem **folderu sa binarnim fajlovima**?

### [Mreža](windows-local-privilege-escalation/#network)

* [ ] Nabrojite mrežu (deljeni resursi, interfejsi, rute, susedi, ...)
* [ ] Posebno obratite pažnju na mrežne servise koji slušaju na lokalnom računaru (127.0.0.1)

### [Windows Kredencijali](windows-local-privilege-escalation/#windows-credentials)

* [ ] [**Winlogon** ](windows-local-privilege-escalation/#winlogon-credentials)kredencijali
* [ ] [**Windows Vault**](windows-local-privilege-escalation/#credentials-manager-windows-vault) kredencijali koje možete koristiti?
* [ ] Interesantni [**DPAPI kredencijali**](windows-local-privilege-escalation/#dpapi)?
* [ ] Lozinke sačuvanih [**Wifi mreža**](windows-local-privilege-escalation/#wifi)?
* [ ] Interesantne informacije u [**sačuvanim RDP konekcijama**](windows-local-privilege-escalation/#saved-rdp-connections)?
* [ ] Lozinke u [**nedavno pokrenutim komandama**](windows-local-privilege-escalation/#recently-run-commands)?
* [ ] [**Upravljač kredencijalima za udaljenu radnu površinu**](windows-local-privilege-escalation/#remote-desktop-credential-manager) lozinke?
* [ ] [**AppCmd.exe** postoji](windows-local-privilege-escalation/#appcmd-exe)? Kredencijali?
* [ ] [**SCClient.exe**](windows-local-privilege-escalation/#scclient-sccm)? Učitavanje DLL-a sa strane?

### [Fajlovi i Registar (Kredencijali)](windows-local-privilege-escalation/#files-and-registry-credentials)

* [ ] **Putty:** [**Kredencijali**](windows-local-privilege-escalation/#putty-creds) **i** [**SSH host ključevi**](windows-local-privilege-escalation/#putty-ssh-host-keys)
* [ ] [**SSH ključevi u registru**](windows-local-privilege-escalation/#ssh-keys-in-registry)?
* [ ] Lozinke u [**neprisutnim fajlovima**](windows-local-privilege-escalation/#unattended-files)?
* [ ] Bilo kakva **SAM & SYSTEM** rezerva?
* [ ] [**Cloud kredencijali**](windows-local-privilege-escalation/#cloud-credentials)?
* [ ] Fajl [**McAfee SiteList.xml**](windows-local-privilege-escalation/#mcafee-sitelist.xml)?
* [ ] [**Keširana GPP lozinka**](windows-local-privilege-escalation/#cached-gpp-pasword)?
* [ ] Lozinka u [**IIS Web konfiguracionom fajlu**](windows-local-privilege-escalation/#iis-web-config)?
* [ ] Interesantne informacije u [**web** **logovima**](windows-local-privilege-escalation/#logs)?
* [ ] Da li želite da **zatražite kredencijale** od korisnika?
* [ ] Interesantni [**fajlovi unutar Recycle Bina**](windows-local-privilege-escalation/#credentials-in-the-recyclebin)?
* [ ] Ostali **registri koji sadrže kredencijale**]\(windows-local-privilege-escalation/#inside-the-registry)?
* [ ] Unutar [**Podataka pretraživača**](windows-local-privilege-escalation/#browsers-history) (baze podataka, istorija, obeleživači, ...)?
* [ ] [**Opšta pretraga lozinki**](windows-local-privilege-escalation/#generic-password-search-in-files-and-registry) u fajlovima i registru
* [ ] [**Alati**](windows-local-privilege-escalation/#tools-that-search-for-passwords) za automatsku pretragu lozinki

### [Procureni Handleri](windows-local-privilege-escalation/#leaked-handlers)

* [ ] Imate li pristup nekom handleru procesa pokrenutog od strane administratora?

### [Imitacija Klijenta Cevi](windows-local-privilege-escalation/#named-pipe-client-impersonation)

* [ ] Proverite da li možete zloupotrebiti to

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
