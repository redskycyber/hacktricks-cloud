# macOS Dangerous Entitlements & TCC perms

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodiču PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

{% hint style="warning" %}
Imajte na umu da ovlašćenja koja počinju sa **`com.apple`** nisu dostupna trećim licima, samo ih Apple može odobriti.
{% endhint %}

## Visoko

### `com.apple.rootless.install.heritable`

Ovlašćenje **`com.apple.rootless.install.heritable`** omogućava **zaobilazak SIP-a**. Proverite [ovde za više informacija](macos-sip.md#com.apple.rootless.install.heritable).

### **`com.apple.rootless.install`**

Ovlašćenje **`com.apple.rootless.install`** omogućava **zaobilazak SIP-a**. Proverite [ovde za više informacija](macos-sip.md#com.apple.rootless.install).

### **`com.apple.system-task-ports` (ranije nazvano `task_for_pid-allow`)**

Ovo ovlašćenje omogućava dobijanje **task porta za bilo** koji proces, osim kernela. Proverite [**ovde za više informacija**](../macos-proces-abuse/macos-ipc-inter-process-communication/).

### `com.apple.security.get-task-allow`

Ovo ovlašćenje omogućava drugim procesima sa ovlašćenjem **`com.apple.security.cs.debugger`** da dobiju task port procesa pokrenutog binarnim fajlom sa ovim ovlašćenjem i **ubace kod u njega**. Proverite [**ovde za više informacija**](../macos-proces-abuse/macos-ipc-inter-process-communication/).

### `com.apple.security.cs.debugger`

Aplikacije sa ovlašćenjem za Alat za Debugovanje mogu pozvati `task_for_pid()` da dobiju validan task port za nepotpisane i treće strane aplikacije sa ovlašćenjem `Get Task Allow` postavljenim na `true`. Međutim, čak i sa ovlašćenjem za alat za debugovanje, debugger **ne može dobiti task portove** procesa koji **nemaju ovlašćenje za Get Task Allow**, i koji su stoga zaštićeni Sistemskom Integritetnom Zaštitom. Proverite [**ovde za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_debugger).

### `com.apple.security.cs.disable-library-validation`

Ovo ovlašćenje omogućava **učitavanje okvira, dodataka ili biblioteka bez potpisa od strane Apple-a ili potpisanog istim Team ID-em** kao glavni izvršni fajl, tako da napadač može zloupotrebiti neko proizvoljno učitavanje biblioteke da ubaci kod. Proverite [**ovde za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-library-validation).

### `com.apple.private.security.clear-library-validation`

Ovo ovlašćenje je vrlo slično **`com.apple.security.cs.disable-library-validation`** ali **umesto** direktnog onemogućavanja validacije biblioteke, omogućava procesu da **pozove `csops` sistemski poziv da je onemogući**.\
Proverite [**ovde za više informacija**](https://theevilbit.github.io/posts/com.apple.private.security.clear-library-validation/).

### `com.apple.security.cs.allow-dyld-environment-variables`

Ovo ovlašćenje omogućava **korišćenje DYLD okruženjskih promenljivih** koje se mogu koristiti za ubacivanje biblioteka i koda. Proverite [**ovde za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables).

### `com.apple.private.tcc.manager` ili `com.apple.rootless.storage`.`TCC`

[**Prema ovom blogu**](https://objective-see.org/blog/blog\_0x4C.html) **i** [**ovom blogu**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/), ova ovlašćenja omogućavaju **modifikaciju** baze podataka **TCC**.

### **`system.install.apple-software`** i **`system.install.apple-software.standar-user`**

Ova ovlašćenja omogućavaju **instaliranje softvera bez traženja dozvole** korisnika, što može biti korisno za **eskaciju privilegija**.

### `com.apple.private.security.kext-management`

Ovlašćenje potrebno za traženje od kernela da učita kernel ekstenziju.

### **`com.apple.private.icloud-account-access`**

Ovlašćenje **`com.apple.private.icloud-account-access`** omogućava komunikaciju sa **`com.apple.iCloudHelper`** XPC servisom koji će **pružiti iCloud tokene**.

**iMovie** i **Garageband** imali su ovo ovlašćenje.

Za više **informacija** o eksploataciji za **dobijanje icloud tokena** iz tog ovlašćenja pogledajte predavanje: [**#OBTS v5.0: "Šta se dešava na vašem Mac-u, ostaje na Apple-ovom iCloud-u?!" - Wojciech Regula**](https://www.youtube.com/watch?v=\_6e2LhmxVc0)

### `com.apple.private.tcc.manager.check-by-audit-token`

TODO: Ne znam šta ovo omogućava

### `com.apple.private.apfs.revert-to-snapshot`

TODO: U [**ovom izveštaju**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **se pominje da bi ovo moglo biti korišćeno** za ažuriranje SSV-zaštićenih sadržaja nakon ponovnog pokretanja. Ako znate kako, pošaljite PR molim vas!

### `com.apple.private.apfs.create-sealed-snapshot`

TODO: U [**ovom izveštaju**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **se pominje da bi ovo moglo biti korišćeno** za ažuriranje SSV-zaštićenih sadržaja nakon ponovnog pokretanja. Ako znate kako, pošaljite PR molim vas!

### `keychain-access-groups`

Ovo ovlašćenje nabraja **grupe ključeva** kojima aplikacija ima pristup:

```xml
<key>keychain-access-groups</key>
<array>
<string>ichat</string>
<string>apple</string>
<string>appleaccount</string>
<string>InternetAccounts</string>
<string>IMCore</string>
</array>
```

### **`kTCCServiceSystemPolicyAllFiles`**

Daje dozvole za **Pristup celom disku**, jednu od najviših dozvola koje možete imati u TCC-u.

### **`kTCCServiceAppleEvents`**

Omogućava aplikaciji slanje događaja drugim aplikacijama koje se često koriste za **automatizaciju zadataka**. Kontrolišući druge aplikacije, može zloupotrebiti dozvole koje su date tim drugim aplikacijama.

Na primer, može ih naterati da zatraže korisnikovu lozinku:

```bash
osascript -e 'tell app "App Store" to activate' -e 'tell app "App Store" to activate' -e 'tell app "App Store" to display dialog "App Store requires your password to continue." & return & return default answer "" with icon 1 with hidden answer with title "App Store Alert"'
```

Ili ih naterati da izvrše **proizvoljne radnje**.

### **`kTCCServiceEndpointSecurityClient`**

Dozvoljava, između ostalih dozvola, da **piše u korisničku TCC bazu podataka**.

### **`kTCCServiceSystemPolicySysAdminFiles`**

Dozvoljava **promenu** atributa **`NFSHomeDirectory`** korisnika koji menja putanju njegove matične fascikle i time omogućava **zaobilazak TCC**.

### **`kTCCServiceSystemPolicyAppBundles`**

Dozvoljava modifikaciju fajlova unutar aplikativnih paketa (unutar app.app), što je **podrazumevano zabranjeno**.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Moguće je proveriti ko ima ovaj pristup u _Sistemskim postavkama_ > _Privatnost & Bezbednost_ > _Upravljanje aplikacijama_.

### `kTCCServiceAccessibility`

Proces će moći da **zloupotrebi macOS funkcije pristupačnosti**, što znači da na primer može da pritisne tasterske prečice. Tako bi mogao da zatraži pristup kontroli aplikacije poput Findera i odobri dijalog sa ovom dozvolom.

## Srednje

### `com.apple.security.cs.allow-jit`

Ova dozvola omogućava da se **kreira memorija koja je upisiva i izvršna** prolaskom `MAP_JIT` zastave ka `mmap()` sistemskoj funkciji. Proverite [**ovde za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-jit).

### `com.apple.security.cs.allow-unsigned-executable-memory`

Ova dozvola omogućava **zamenu ili zakrpu C koda**, korišćenje dugo zastarelog **`NSCreateObjectFileImageFromMemory`** (što je fundamentalno nesigurno), ili korišćenje **DVDPlayback** okvira. Proverite [**ovde za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-unsigned-executable-memory).

{% hint style="danger" %}
Uključivanje ove dozvole izlaže vašu aplikaciju uobičajenim ranjivostima u jezicima sa kodom koji nije siguran za memoriju. Pažljivo razmislite da li vaša aplikacija zahteva ovaj izuzetak.
{% endhint %}

### `com.apple.security.cs.disable-executable-page-protection`

Ova dozvola omogućava da se **modifikuju sekcije sopstvenih izvršnih fajlova** na disku kako bi se silom izašlo. Proverite [**ovde za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-executable-page-protection).

{% hint style="danger" %}
Dozvola za Onemogućavanje Zaštite Izvršne Memorije je ekstremna dozvola koja uklanja osnovnu sigurnosnu zaštitu iz vaše aplikacije, čime se omogućava napadaču da prepiše izvršni kod vaše aplikacije bez otkrivanja. Preferirajte uže dozvole ako je moguće.
{% endhint %}

### `com.apple.security.cs.allow-relative-library-loads`

TODO

### `com.apple.private.nullfs_allow`

Ova dozvola omogućava montiranje nullfs fajl sistema (podrazumevano zabranjeno). Alat: [**mount\_nullfs**](https://github.com/JamaicanMoose/mount\_nullfs/tree/master).

### `kTCCServiceAll`

Prema ovom blog postu, ova TCC dozvola obično se nalazi u obliku:

```
[Key] com.apple.private.tcc.allow-prompting
[Value]
[Array]
[String] kTCCServiceAll
```

Dozvoli procesu da **zatraži sve TCC dozvole**.

### **`kTCCServicePostEvent`**

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
