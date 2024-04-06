# Skeleton Key

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Napad Skeleton Key

Napad **Skeleton Key** je sofisticirana tehnika koja omogućava napadačima da **zaobiđu autentifikaciju Active Directory-ja** ubacivanjem glavne lozinke u kontroler domena. Ovo omogućava napadaču da se **autentifikuje kao bilo koji korisnik** bez njihove lozinke, efektivno im **dajući neograničen pristup** domenu.

Može se izvesti pomoću alata [Mimikatz](https://github.com/gentilkiwi/mimikatz). Da bi se izveo ovaj napad, **neophodna su administratorska prava domena**, a napadač mora ciljati svaki kontroler domena kako bi osigurao sveobuhvatno probijanje. Međutim, efekat napada je privremen, jer **ponovno pokretanje kontrolera domena uklanja malver**, što zahteva ponovnu implementaciju za trajni pristup.

**Izvršavanje napada** zahteva jednu komandu: `misc::skeleton`.

## Mere zaštite

Strategije zaštite od ovakvih napada uključuju praćenje određenih ID-ova događaja koji ukazuju na instalaciju servisa ili korišćenje osetljivih privilegija. Konkretno, traženje ID-a događaja sistema 7045 ili ID-a događaja bezbednosti 4673 može otkriti sumnjive aktivnosti. Dodatno, pokretanje `lsass.exe` kao zaštićenog procesa može značajno ometati napore napadača, jer zahteva korišćenje drajvera u režimu jezgra, što povećava složenost napada.

Evo PowerShell komandi za poboljšanje sigurnosnih mera:

* Za otkrivanje instalacije sumnjivih servisa koristite: `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*"}`
* Konkretno, za otkrivanje Mimikatz-ovog drajvera, može se koristiti sledeća komanda: `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*" -and $_.message -like "*mimidrv*"}`
* Za ojačavanje `lsass.exe`, preporučuje se omogućavanje kao zaštićenog procesa: `New-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa -Name RunAsPPL -Value 1 -Verbose`

Provera nakon ponovnog pokretanja sistema je ključna kako bi se osiguralo da su zaštitne mere uspešno primenjene. To se može postići pomoću: `Get-WinEvent -FilterHashtable @{Logname='System';ID=12} | ?{$_.message -like "*protected process*`

## Reference

* [https://blog.netwrix.com/2022/11/29/skeleton-key-attack-active-directory/](https://blog.netwrix.com/2022/11/29/skeleton-key-attack-active-directory/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
