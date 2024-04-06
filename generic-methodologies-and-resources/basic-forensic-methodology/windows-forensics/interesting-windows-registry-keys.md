# Interesting Windows Registry Keys

### Interesantni Windows registarski ključevi

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### **Windows verzija i informacije o vlasniku**

* Na lokaciji **`Software\Microsoft\Windows NT\CurrentVersion`**, možete pronaći Windows verziju, Service Pack, vreme instalacije i ime registrovanog vlasnika na jednostavan način.

### **Ime računara**

* Hostname se nalazi pod **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Podešavanje vremenske zone**

* Vremenska zona sistema se čuva u **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Pracenje vremena pristupa**

* Prema zadanim postavkama, praćenje vremena poslednjeg pristupa je isključeno (**`NtfsDisableLastAccessUpdate=1`**). Da biste ga omogućili, koristite: `fsutil behavior set disablelastaccess 0`

### Windows verzije i Service Pack-ovi

* **Windows verzija** označava izdanje (npr. Home, Pro) i njegovo izdanje (npr. Windows 10, Windows 11), dok su **Service Pack-ovi** ažuriranja koja uključuju ispravke i ponekad nove funkcije.

### Omogućavanje praćenja vremena poslednjeg pristupa

* Omogućavanje praćenja vremena poslednjeg pristupa omogućava vam da vidite kada su datoteke poslednji put otvorene, što može biti ključno za forenzičku analizu ili praćenje sistema.

### Detalji o mrežnim informacijama

* Registar sadrži obimne podatke o mrežnim konfiguracijama, uključujući **vrste mreža (bežične, kablovske, 3G)** i **kategorije mreže (Javna, Privatna/Kućna, Domen/Posao)**, što je važno za razumevanje postavki mrežne sigurnosti i dozvola.

### Klijentsko keširanje (CSC)

* **CSC** poboljšava pristup datotekama van mreže keširanjem kopija deljenih datoteka. Različite postavke **CSCFlags** kontrolišu način i koje datoteke se keširaju, što utiče na performanse i korisničko iskustvo, posebno u okruženjima sa povremenom konekcijom.

### Programi koji se automatski pokreću

* Programi navedeni u različitim registarskim ključevima `Run` i `RunOnce` automatski se pokreću prilikom pokretanja sistema, utičući na vreme pokretanja sistema i potencijalno predstavljajući tačke interesa za identifikaciju malvera ili neželjenog softvera.

### Shellbags

* **Shellbags** ne samo da čuvaju postavke za prikaz fascikli, već pružaju i forenzičke dokaze o pristupu fasciklama čak i ako fascikla više ne postoji. Oni su neprocenjivi za istrage, otkrivajući korisničku aktivnost koja nije očigledna na druge načine.

### Informacije i forenzika o USB uređajima

* Detalji o USB uređajima koji se čuvaju u registru mogu pomoći u praćenju koji su uređaji bili povezani sa računarom, potencijalno povezujući uređaj sa prenosom osetljivih datoteka ili incidentima neovlašćenog pristupa.

### Serijski broj zapremine

* **Serijski broj zapremine** može biti ključan za praćenje specifične instance sistema datoteka, korisno u forenzičkim scenarijima gde je potrebno utvrditi poreklo datoteke na različitim uređajima.

### **Detalji o isključivanju**

* Vreme isključivanja i broj isključivanja (samo za XP) čuvaju se u **`System\ControlSet001\Control\Windows`** i **`System\ControlSet001\Control\Watchdog\Display`**.

### **Konfiguracija mreže**

* Za detaljne informacije o mrežnom interfejsu, pogledajte **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
* Prva i poslednja vremena povezivanja na mrežu, uključujući VPN veze, beleže se pod različitim putanjama u **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`**.

### **Deljene fascikle**

* Deljene fascikle i postavke nalaze se pod **`System\ControlSet001\Services\lanmanserver\Shares`**. Postavke klijentskog keširanja (CSC) određuju dostupnost datoteka van mreže.

### **Programi koji se automatski pokreću**

* Putanje poput **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** i slični unosi pod `Software\Microsoft\Windows\CurrentVersion` detaljno opisuju programe koji se pokreću prilikom pokretanja sistema.

### **Pretrage i uneti putovi**

* Pretrage i uneti putovi u Explorer-u se prate u registru pod **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** za WordwheelQuery i TypedPaths, redom.

### **Nedavni dokumenti i Office datoteke**

* Nedavno korišćeni dokumenti i Office datoteke se beleže u `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` i specifičnim putanjama za verzije Office-a.

### **Najskorije korišćene (MRU) stavke**

* Liste MRU, koje pokazuju nedavne putanje do datoteka i komande, čuvaju se u različitim podključevima `ComDlg32` i `Explorer` pod `NTUSER.DAT`.

### **Pracenje korisničke aktivnosti**

* Funkcija User Assist beleži detaljne statistike o korišćenju aplikacija, uključujući broj pokretanja i vreme poslednjeg pokretanja, na lokaciji **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Analiza Shellbags-a**

* Shellbags, koji otkrivaju detalje o pristupu fasciklama
