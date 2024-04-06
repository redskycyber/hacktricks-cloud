# WmicExec

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Kako radi objašnjeno

Procesi se mogu otvoriti na hostovima gde su poznato korisničko ime i ili lozinka ili heš, korišćenjem WMI-ja. Komande se izvršavaju korišćenjem WMI-ja putem Wmiexec-a, pružajući polu-interaktivno iskustvo ljuske.

**dcomexec.py:** Koristeći različite DCOM endpointe, ovaj skript nudi polu-interaktivnu ljusku sličnu wmiexec.py, posebno koristeći ShellBrowserWindow DCOM objekat. Trenutno podržava MMC20. Aplikaciju, Shell prozore i Shell Browser prozore. (izvor: [Hacking Articles](https://www.hackingarticles.in/beginners-guide-to-impacket-tool-kit-part-1/))

## Osnove WMI-ja

### Namespace

Strukturiran u hijerarhiji sličnoj direktorijumima, vrhunski kontejner WMI-ja je \root, ispod kojeg su organizovani dodatni direktorijumi, nazvani namespace-ovi. Komande za listanje namespace-ova:

```bash
# Retrieval of Root namespaces
gwmi -namespace "root" -Class "__Namespace" | Select Name

# Enumeration of all namespaces (administrator privileges may be required)
Get-WmiObject -Class "__Namespace" -Namespace "Root" -List -Recurse 2> $null | select __Namespace | sort __Namespace

# Listing of namespaces within "root\cimv2"
Get-WmiObject -Class "__Namespace" -Namespace "root\cimv2" -List -Recurse 2> $null | select __Namespace | sort __Namespace
```

Klase unutar namespace-a mogu biti navedene koristeći:

```bash
gwmwi -List -Recurse # Defaults to "root\cimv2" if no namespace specified
gwmi -Namespace "root/microsoft" -List -Recurse
```

### **Klase**

Poznavanje imena WMI klase, kao što je win32\_process, i namespace-a u kojem se nalazi je ključno za bilo koju WMI operaciju. Komande za listanje klasa koje počinju sa `win32`:

```bash
Get-WmiObject -Recurse -List -class win32* | more # Defaults to "root\cimv2"
gwmi -Namespace "root/microsoft" -List -Recurse -Class "MSFT_MpComput*"
```

Pozivanje klase:

```bash
# Defaults to "root/cimv2" when namespace isn't specified
Get-WmiObject -Class win32_share
Get-WmiObject -Namespace "root/microsoft/windows/defender" -Class MSFT_MpComputerStatus
```

### Metode

Metode, koje su jedna ili više izvršnih funkcija WMI klasa, mogu biti izvršene.

```bash
# Class loading, method listing, and execution
$c = [wmiclass]"win32_share"
$c.methods
# To create a share: $c.Create("c:\share\path","name",0,$null,"My Description")
```

```bash
# Method listing and invocation
Invoke-WmiMethod -Class win32_share -Name Create -ArgumentList @($null, "Description", $null, "Name", $null, "c:\share\path",0)
```

## WMI Enumeracija

### Status WMI servisa

Komande za proveru da li je WMI servis operativan:

```bash
# WMI service status check
Get-Service Winmgmt

# Via CMD
net start | findstr "Instrumentation"
```

### Informacije o sistemu i procesima

Prikupljanje informacija o sistemu i procesima putem WMI-a:

```bash
Get-WmiObject -ClassName win32_operatingsystem | select * | more
Get-WmiObject win32_process | Select Name, Processid
```

Za napadače, WMI je moćan alat za nabavku osetljivih podataka o sistemima ili domenima.

```bash
wmic computerystem list full /format:list
wmic process list /format:list
wmic ntdomain list /format:list
wmic useraccount list /format:list
wmic group list /format:list
wmic sysaccount list /format:list
```

### **Ručno udaljeno WMI upitivanje**

Moguće je tajno identifikovati lokalne administratore na udaljenom računaru i prijavljene korisnike putem određenih WMI upita. `wmic` takođe podržava čitanje iz tekstualne datoteke radi izvršavanja komandi na više čvorova istovremeno.

Da biste udaljeno izvršili proces putem WMI, kao što je implementacija Empire agenta, koristi se sledeća struktura komande, pri čemu uspešno izvršavanje pokazuje povratnu vrednost "0":

```bash
wmic /node:hostname /user:user path win32_process call create "empire launcher string here"
```

Ovaj proces ilustruje mogućnost WMI-a za udaljeno izvršavanje i enumeraciju sistema, ističući njegovu korisnost kako za administraciju sistema, tako i za testiranje penetracije.

## Reference

* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-3-wmi-and-winrm/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

## Automatski alati

* [**SharpLateral**](https://github.com/mertdas/SharpLateral):

{% code overflow="wrap" %}
```bash
SharpLateral redwmi HOSTNAME C:\\Users\\Administrator\\Desktop\\malware.exe
```
{% endcode %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
