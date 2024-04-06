# Escaping from KIOSKs

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite videti **oglašavanje vaše kompanije na HackTricks-u** ili **preuzeti HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Provera mogućih akcija unutar GUI aplikacije

**Uobičajeni dijalozi** su opcije **čuvanja fajla**, **otvaranja fajla**, izbora fonta, boje... Većina njih će **ponuditi punu funkcionalnost Explorer-a**. To znači da ćete moći pristupiti funkcionalnostima Explorer-a ako možete pristupiti ovim opcijama:

* Zatvori/Zatvori kao
* Otvori/Otvori sa
* Štampaj
* Izvoz/Uvoz
* Pretraga
* Skeniranje

Treba da proverite da li možete:

* Izmeniti ili kreirati nove fajlove
* Kreirati simboličke linkove
* Pristupiti ograničenim područjima
* Izvršiti druge aplikacije

### Izvršavanje komandi

Možda **koristeći opciju `Otvori sa`** možete otvoriti/izvršiti neku vrstu shell-a.

#### Windows

Na primer _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ pronađite više binarnih fajlova koji se mogu koristiti za izvršavanje komandi (i izvršavanje neočekivanih akcija) ovde: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

#### \*NIX \_\_

_bash, sh, zsh..._ Više ovde: [https://gtfobins.github.io/](https://gtfobins.github.io)

## Windows

### Zaobilaženje restrikcija putanje

* **Okružne promenljive**: Postoji mnogo okružnih promenljivih koje pokazuju na neku putanju
* **Drugi protokoli**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Simbolički linkovi**
* **Prečice**: CTRL+N (otvori novu sesiju), CTRL+R (Izvrši komande), CTRL+SHIFT+ESC (Upravitelj zadataka), Windows+E (otvori explorer), CTRL-B, CTRL-I (Omiljeni), CTRL-H (Istorija), CTRL-L, CTRL-O (Dijalog za otvaranje fajla), CTRL-P (Dijalog za štampanje), CTRL-S (Sačuvaj kao)
* Skriveni administratorski meni: CTRL-ALT-F8, CTRL-ESC-F9
* **Shell URI-ji**: _shell:Administrative Tools, shell:DocumentsLibrary, shell:Librariesshell:UserProfiles, shell:Personal, shell:SearchHomeFolder, shell:Systemshell:NetworkPlacesFolder, shell:SendTo, shell:UsersProfiles, shell:Common Administrative Tools, shell:MyComputerFolder, shell:InternetFolder_
* **UNC putanje**: Putanje za povezivanje sa deljenim fasciklama. Trebali biste pokušati da se povežete sa C$ lokalne mašine ("\\\127.0.0.1\c$\Windows\System32")
* **Još UNC putanja:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOMEPATH%     | %LOCALAPPDATA%       |
| %LOGONSERVER%             | %PATH%         | %PATHEXT%            |
| %ProgramData%             | %ProgramFiles% | %ProgramFiles(x86)%  |
| %PROMPT%                  | %PSModulePath% | %Public%             |
| %SYSTEMDRIVE%             | %SYSTEMROOT%   | %TEMP%               |
| %TMP%                     | %USERDOMAIN%   | %USERNAME%           |
| %USERPROFILE%             | %WINDIR%       |                      |

### Preuzmite svoje binarne fajlove

Console: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Explorer: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Registry editor: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

### Pristupanje fajl sistemu preko browser-a

| PUTANJA             | PUTANJA           | PUTANJA            | PUTANJA             |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |

### Prečice

* Lepkaste tipke - Pritisnite SHIFT 5 puta
* Miš tastere - SHIFT+ALT+NUMLOCK
* Visok kontrast - SHIFT+ALT+PRINTSCN
* Prekidač tastere - Držite NUMLOCK 5 sekundi
* Filter tastere - Držite desni SHIFT 12 sekundi
* WINDOWS+F1 - Windows pretraga
* WINDOWS+D - Prikaz radne površine
* WINDOWS+E - Pokreni Windows Explorer
* WINDOWS+R - Pokreni
* WINDOWS+U - Centar za olakšavanje pristupa
* WINDOWS+F - Pretraga
* SHIFT+F10 - Kontekstni meni
* CTRL+SHIFT+ESC - Upravitelj zadataka
* CTRL+ALT+DEL - Ekran za prijavu na novijim verzijama Windows-a
* F1 - Pomoć F3 - Pretraga
* F6 - Traka adrese
* F11 - Uključivanje/isključivanje prikaza preko celog ekrana u Internet Explorer-u
* CTRL+H - Istorija Internet Explorer-a
* CTRL+T - Internet Explorer - Novi tab
* CTRL+N - Internet Explorer - Nova stranica
* CTRL+O - Otvori fajl
* CTRL+S - Sačuvaj CTRL+N - Novi RDP / Citrix

### Swajpovi

* Swajp s leve strane na desnu da biste videli sve otvorene prozore, minimizirajući KIOSK aplikaciju i pristupajući celom operativnom sistemu direktno;
* Swajp s desne strane na levu da biste otvorili Action Center, minimizirajući KIOSK aplikaciju i pristupajući celom operativnom sistemu direktno;
* Swajp od vrha ekrana da biste videli traku sa naslovom za aplikaciju otvorenu u režimu punog ekrana;
* Swajp od dna ekrana da biste prikazali traku sa zadacima u aplikaciji u punom ekranu.

### Trikovi za Internet Explorer

#### 'Image Toolbar'

To je traka sa alatkama koja se pojavljuje na gornjem levom delu slike kada se klikne na nju. Moći ćete da sačuvate, odštampate, pošaljete e-poštu, otvorite "Moje slike" u Exploreru. Kiosk mora koristiti Internet Explorer.

#### Shell Protocol

Unesite ove URL-ove da biste dobili prikaz Explorer-a:

* `shell:Administrative Tools`
* `shell:DocumentsLibrary`
* `shell:Libraries`
* `shell:UserProfiles`
* `shell:Personal`
* `shell:SearchHomeFolder`
* `shell:NetworkPlacesFolder`
* `shell:SendTo`
* `shell:UserProfiles`
* `shell:Common Administrative Tools`
* `shell:MyComputerFolder`
* `shell:InternetFolder`
* `Shell:Profile`
* `Shell:ProgramFiles`
* `Shell:System`
* `Shell:ControlPanelFolder`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> Control Panel
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> My Computer
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> My Network Places
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> Internet Explorer

### Prikazivanje ekstenzija fajlova

Proverite ovu stranicu za više informacija: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

## Trikovi za pretraživače

Rezervne verzije iKat-a:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\\

Kreirajte zajednički dijalog koristeći JavaScript i pristupite istraživaču fajlova: `document.write('<input/type=file>')` Izvor: https://medium.com/@Rend\_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

## iPad

### Gestovi i dugmad

* Swajp nagore sa četiri (ili pet) prsta / Dvoklik na dugme Home: Da biste videli prikaz više zadataka i promenili aplikaciju
* Swajp na jednu ili drugu stranu sa četiri ili pet prstiju: Da biste promenili na sledeću/prethodnu aplikaciju
* Štipanje ekrana sa pet prstiju / Dodir dugmeta Home / Swajp nagore jednim prstom sa dna ekrana brzim pokretom prema gore: Da biste pristupili Početnom ekranu
* Swajp jednim prstom sa dna ekrana samo 1-2 inča (sporo): Pojavljuje se dock
* Swajp nadole sa vrha ekrana jednim prstom: Da biste videli obaveštenja
* Swajp nadole jednim prstom u gornjem desnom uglu ekrana: Da biste videli kontrolni centar iPad Pro-a
* Swajp jednim prstom sa leve strane ekrana 1-2 inča: Da biste videli današnji prikaz
* Brzi swajp jednim prstom sa centra ekrana udesno ili ulevo: Da biste promenili na sledeću/prethodnu aplikaciju
* Pritisnite i držite dugme On/**Off**/Sleep u gornjem desnom uglu **iPad +** Pomerite klizač Slide to **power off** sve do kraja udesno: Da biste isključili napajanje
* Pritisnite dugme On/**Off**/Sleep u gornjem desnom uglu **iPad-a i dugme Home nekoliko sekundi**: Da biste prinudno isključili napajanje
* Pritisnite dugme On/**Off**/Sleep u gornjem desnom uglu **iPad-a i dugme Home brzo**: Da biste napravili snimak ekrana koji će se pojaviti u donjem levom uglu ekrana. Pritisnite oba dugmeta istovremeno veoma kratko, jer ako ih držite nekoliko sekundi, izvršiće se prinudno isključivanje napajanja.

### Prečice

Treba vam tastatura za iPad ili adapter za USB tastaturu. Ovde će biti prikazane samo prečice koje mogu pomoći pri izlasku iz aplikacije.

| Taster | Naziv           |
| ------ | --------------- |
| ⌘      | Komanda         |
| ⌥      | Opcija (Alt)    |
| ⇧      | Shift           |
| ↩      | Povratak        |
| ⇥      | Tab             |
| ^      | Kontrola        |
| ←      | Leva strelica   |
| →      | Desna strelica  |
| ↑      | Gornja strelica |
| ↓      | Donja strelica  |

#### Sistemske prečice

Ove prečice su za vizuelna podešavanja i podešavanja zvuka, u zavisnosti od upotrebe iPada.

| Prečica | Radnja                                                                                  |
| ------- | --------------------------------------------------------------------------------------- |
| F1      | Smanji osvetljenost ekrana                                                              |
| F2      | Povećaj osvetljenost ekrana                                                             |
| F7      | Nazad jedna pesma                                                                       |
| F8      | Pusti/pauziraj                                                                          |
| F9      | Preskoči pesmu                                                                          |
| F10     | Isključi zvuk                                                                           |
| F11     | Smanji jačinu zvuka                                                                     |
| F12     | Povećaj jačinu zvuka                                                                    |
| ⌘ Space | Prikazuje listu dostupnih jezika; da biste izabrali jedan, ponovo dodirnite razmaknicu. |

#### Navigacija na iPad-u

| Prečica                                                    | Radnja                                                          |
| ---------------------------------------------------------- | --------------------------------------------------------------- |
| ⌘H                                                         | Idi na Početni ekran                                            |
| ⌘⇧H (Command-Shift-H)                                      | Idi na Početni ekran                                            |
| ⌘ (Space)                                                  | Otvori Spotlight                                                |
| ⌘⇥ (Command-Tab)                                           | Prikazuje poslednjih deset korišćenih aplikacija                |
| ⌘\~                                                        | Idi na poslednju aplikaciju                                     |
| ⌘⇧3 (Command-Shift-3)                                      | Snimak ekrana (lebdi u donjem levom uglu za čuvanje ili radnju) |
| ⌘⇧4                                                        | Snimak ekrana i otvori ga u editoru                             |
| Pritisnite i držite ⌘                                      | Lista dostupnih prečica za aplikaciju                           |
| ⌘⌥D (Command-Option/Alt-D)                                 | Prikazuje dock                                                  |
| ^⌥H (Control-Option-H)                                     | Dugme Početni ekran                                             |
| ^⌥H H (Control-Option-H-H)                                 | Prikazuje traku sa više zadataka                                |
| ^⌥I (Control-Option-i)                                     | Biranje stavke                                                  |
| Escape                                                     | Dugme Nazad                                                     |
| → (Desna strelica)                                         | Sledeća stavka                                                  |
| ← (Leva strelica)                                          | Prethodna stavka                                                |
| ↑↓ (Gornja strelica, Donja strelica)                       | Istovremeno dodirnite izabranu stavku                           |
| ⌥ ↓ (Opcija-Dole)                                          | Pomeranje nadole                                                |
| ⌥↑ (Opcija-Gore)                                           | Pomeranje nagore                                                |
| ⌥← ili ⌥→ (Opcija-Leva strelica ili Opcija-Desna strelica) | Pomeranje ulevo ili udesno                                      |
| ^⌥S (Control-Option-S)                                     | Uključivanje ili isključivanje govora VoiceOver                 |
| ⌘⇧⇥ (Command-Shift-Tab)                                    | Prebacivanje na prethodnu aplikaciju                            |
| ⌘⇥ (Command-Tab)                                           | Vraćanje na originalnu aplikaciju                               |
| ←+→, zatim O                                               |                                                                 |
| ### Prečice u Safariju                                     |                                                                 |

| Prečica                 | Radnja                                                        |
| ----------------------- | ------------------------------------------------------------- |
| ⌘L (Command-L)          | Otvori lokaciju                                               |
| ⌘T                      | Otvori novi tab                                               |
| ⌘W                      | Zatvori trenutni tab                                          |
| ⌘R                      | Osvježi trenutni tab                                          |
| ⌘.                      | Zaustavi učitavanje trenutnog taba                            |
| ^⇥                      | Prebaci se na sljedeći tab                                    |
| ^⇧⇥ (Control-Shift-Tab) | Prebaci se na prethodni tab                                   |
| ⌘L                      | Odaberi tekstualni unos/URL polje za izmjenu                  |
| ⌘⇧T (Command-Shift-T)   | Otvori posljednje zatvoreni tab (može se koristiti više puta) |
| ⌘\[                     | Vrati se jednu stranicu unazad u povijesti pregledavanja      |
| ⌘]                      | Idi jednu stranicu unaprijed u povijesti pregledavanja        |
| ⌘⇧R                     | Aktiviraj način čitača                                        |

#### Prečice u Mailu

| Prečica                    | Radnja                             |
| -------------------------- | ---------------------------------- |
| ⌘L                         | Otvori lokaciju                    |
| ⌘T                         | Otvori novi tab                    |
| ⌘W                         | Zatvori trenutni tab               |
| ⌘R                         | Osvježi trenutni tab               |
| ⌘.                         | Zaustavi učitavanje trenutnog taba |
| ⌘⌥F (Command-Option/Alt-F) | Pretraži svoj poštanski sandučić   |

## Reference

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite vidjeti **oglašavanje vaše kompanije na HackTricks-u** ili **preuzeti HackTricks u PDF formatu** Provjerite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podijelite svoje hakirajuće trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
