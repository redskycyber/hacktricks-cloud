# Volatility - CheatSheet

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji kibernetički događaj u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je ključno mesto susreta tehnoloških i kibernetičkih profesionalaca u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

Ako želite nešto **brzo i ludo** što će pokrenuti nekoliko Volatility pluginova paralelno, možete koristiti: [https://github.com/carlospolop/autoVolatility](https://github.com/carlospolop/autoVolatility)

```bash
python autoVolatility.py -f MEMFILE -d OUT_DIRECTORY -e /home/user/tools/volatility/vol.py # It will use the most important plugins (could use a lot of space depending on the size of the memory)
```

## Instalacija

### volatility3

```bash
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
python3 setup.py install
python3 vol.py —h
```

#### Metodologija 1

1. **Identifikacija profila**: Pokrenite `volatility2` sa opcijom `imageinfo` kako biste identifikovali profil memorije.

```plaintext
volatility2 -f dump.mem imageinfo
```

2. **Izdvajanje procesa**: Koristite opciju `pslist` da biste izdvojili sve procese iz memorije.

```plaintext
volatility2 -f dump.mem --profile=profil pslist
```

3. **Analiza procesa**: Analizirajte izdvojene procese kako biste pronašli sumnjive aktivnosti ili tragove napada.

```plaintext
volatility2 -f dump.mem --profile=profil procanalysis -p PID
```

4. **Analiza mreže**: Koristite opciju `netscan` da biste pronašli otvorene mrežne veze i aktivnosti.

```plaintext
volatility2 -f dump.mem --profile=profil netscan
```

5. **Analiza registra**: Koristite opciju `hivelist` da biste pronašli registarske ključeve u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil hivelist
```

6. **Analiza datoteka**: Koristite opciju `filescan` da biste pronašli otvorene datoteke u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil filescan
```

7. **Analiza servisa**: Koristite opciju `svcscan` da biste pronašli pokrenute servise u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil svcscan
```

8. **Analiza drajvera**: Koristite opciju `driverirp` da biste pronašli drajvere u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil driverirp
```

9. **Analiza modula**: Koristite opciju `modscan` da biste pronašli učitane module u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil modscan
```

10. **Analiza tokena**: Koristite opciju `tokens` da biste pronašli informacije o tokenima u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil tokens
```

11. **Analiza procesa učitavanja**: Koristite opciju `ldrmodules` da biste pronašli informacije o procesima učitavanja u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil ldrmodules
```

12. **Analiza datoteka učitavanja**: Koristite opciju `ldrmodules` sa dodatnom opcijom `--dump` da biste izdvojili datoteke učitavanja iz memorije.

```plaintext
volatility2 -f dump.mem --profile=profil ldrmodules --dump
```

13. **Analiza heša**: Koristite opciju `hashdump` da biste pronašli heširane lozinke u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil hashdump
```

14. **Analiza šifri**: Koristite opciju `mimikatz` da biste pronašli šifrovane podatke u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil mimikatz
```

15. **Analiza registra**: Koristite opciju `printkey` da biste pronašli vrednosti registra u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil printkey -K "RegistryKey"
```

16. **Analiza događaja**: Koristite opciju `evnets` da biste pronašli informacije o događajima u memoriji.

```plaintext
volatility2 -f dump.mem --profile=profil events
```

17. **Analiza fajlova**: Koristite opciju `filescan` sa dodatnom opcijom `--dump-dir` da biste izdvojili fajlove iz memorije.

```plaintext
volatility2 -f dump.mem --profile=profil filescan --dump-dir=/putanja/do/direktorijuma
```

18. **Analiza registra**: Koristite opciju `printkey` sa dodatnom opcijom `--dump-dir` da biste izdvojili vrednosti registra iz memorije.

```plaintext
volatility2 -f dump.mem --profile=profil printkey -K "RegistryKey" --dump-dir=/putanja/do/direktorijuma
```

```
Download the executable from https://www.volatilityfoundation.org/26
```

```bash
git clone https://github.com/volatilityfoundation/volatility.git
cd volatility
python setup.py install
```

## Komande za Volatility

Pristupite zvaničnoj dokumentaciji na [Volatility command reference](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#kdbgscan)

### Napomena o "list" vs. "scan" pluginovima

Volatility ima dva glavna pristupa pluginovima, što se ponekad odražava u njihovim imenima. "list" pluginovi će pokušati da se kreću kroz strukture Windows Kernela kako bi dobili informacije poput procesa (lociraju i prolaze kroz povezanu listu struktura `_EPROCESS` u memoriji), OS handle-ova (lociranje i listanje tabele handle-ova, dereferenciranje bilo kojih pronađenih pokazivača, itd). Oni se ponašaju slično kao Windows API kada se zatraži, na primer, lista procesa.

To čini "list" pluginove prilično brzim, ali jednako ranjivim kao i Windows API na manipulaciju od strane malvera. Na primer, ako malver koristi DKOM da odvoji proces od povezane liste `_EPROCESS`, neće se prikazati u Task Manageru, niti u pslistu.

"scan" pluginovi, s druge strane, će pristupiti memoriji na način sličan izdvajanju stvari koje bi imale smisla kada bi se dereferencirale kao određene strukture. Na primer, `psscan` će čitati memoriju i pokušati da napravi objekte `_EPROCESS` od nje (koristi skeniranje pool-tagova, što je traženje 4-bajtnih nizova koji ukazuju na prisustvo strukture od interesa). Prednost je u tome što može pronaći procese koji su završili, i čak i ako malver manipuliše povezanom listom `_EPROCESS`, plugin će i dalje pronaći strukturu koja se nalazi u memoriji (jer još uvek mora postojati da bi proces radio). Mana je što su "scan" pluginovi malo sporiji od "list" pluginova, i ponekad mogu dati lažne pozitivne rezultate (proces koji je završio pre dugo vremena i čiji su delovi strukture prepisani drugim operacijama).

Izvor: [http://tomchop.me/2016/11/21/tutorial-volatility-plugins-malware-analysis/](http://tomchop.me/2016/11/21/tutorial-volatility-plugins-malware-analysis/)

## OS Profili

### Volatility3

Kao što je objašnjeno u readme datoteci, trebate staviti **tabelu simbola OS-a** koji želite podržati unutar _volatility3/volatility/symbols_.\
Paketi tabela simbola za različite operativne sisteme dostupni su za **preuzimanje** na:

* [https://downloads.volatilityfoundation.org/volatility3/symbols/windows.zip](https://downloads.volatilityfoundation.org/volatility3/symbols/windows.zip)
* [https://downloads.volatilityfoundation.org/volatility3/symbols/mac.zip](https://downloads.volatilityfoundation.org/volatility3/symbols/mac.zip)
* [https://downloads.volatilityfoundation.org/volatility3/symbols/linux.zip](https://downloads.volatilityfoundation.org/volatility3/symbols/linux.zip)

### Volatility2

#### Spoljni profil

Možete dobiti listu podržanih profila koristeći:

```bash
./volatility_2.6_lin64_standalone --info | grep "Profile"
```

Ako želite koristiti **novi profil koji ste preuzeli** (na primer, linux profil), morate negde kreirati sledeću strukturu foldera: _plugins/overlays/linux_ i staviti unutar ovog foldera zip datoteku koja sadrži profil. Zatim, dobijte broj profila koristeći:

```bash
./vol --plugins=/home/kali/Desktop/ctfs/final/plugins --info
Volatility Foundation Volatility Framework 2.6


Profiles
--------
LinuxCentOS7_3_10_0-123_el7_x86_64_profilex64 - A Profile for Linux CentOS7_3.10.0-123.el7.x86_64_profile x64
VistaSP0x64                                   - A Profile for Windows Vista SP0 x64
VistaSP0x86                                   - A Profile for Windows Vista SP0 x86
```

Možete **preuzeti profile za Linux i Mac** sa [https://github.com/volatilityfoundation/profiles](https://github.com/volatilityfoundation/profiles)

U prethodnom odeljku možete videti da se profil naziva `LinuxCentOS7_3_10_0-123_el7_x86_64_profilex64`, i možete ga koristiti da izvršite nešto poput:

```bash
./vol -f file.dmp --plugins=. --profile=LinuxCentOS7_3_10_0-123_el7_x86_64_profilex64 linux_netscan
```

#### Otkrivanje profila

```bash
volatility -f <memory_dump> imageinfo
```

Ova komanda će vam pomoći da otkrijete informacije o profilu memorije.

```
volatility imageinfo -f file.dmp
volatility kdbgscan -f file.dmp
```

#### **Razlike između imageinfo i kdbgscan**

[**Odavde**](https://www.andreafortuna.org/2017/06/25/volatility-my-own-cheatsheet-part-1-image-identification/): Za razliku od imageinfo koji samo pruža predloge profila, **kdbgscan** je dizajniran da pozitivno identifikuje tačan profil i tačnu KDBG adresu (ako postoji više njih). Ovaj dodatak skenira potpise KDBGHeadera povezane sa Volatility profilima i primenjuje provere ispravnosti kako bi se smanjio broj lažnih pozitiva. Opširnost izlaza i broj provera ispravnosti koje se mogu izvršiti zavise od toga da li Volatility može pronaći DTB, pa ako već znate tačan profil (ili ako imate predlog profila od imageinfo), pobrinite se da ga koristite od .

Uvek pogledajte **broj procesa koje je kdbgscan pronašao**. Ponekad imageinfo i kdbgscan mogu pronaći **više od jednog** odgovarajućeg **profila**, ali samo **validan profil će imati neke procese povezane** (To je zato što je za izdvajanje procesa potrebna tačna KDBG adresa).

```bash
# GOOD
PsActiveProcessHead           : 0xfffff800011977f0 (37 processes)
PsLoadedModuleList            : 0xfffff8000119aae0 (116 modules)
```

```bash
# BAD
PsActiveProcessHead           : 0xfffff800011947f0 (0 processes)
PsLoadedModuleList            : 0xfffff80001197ac0 (0 modules)
```

#### KDBG

**Kernel debugger blok**, poznat kao **KDBG** u Volatility-u, ključan je za forenzičke zadatke koje obavlja Volatility i razni debuggeri. Identifikovan kao `KdDebuggerDataBlock` i tipa `_KDDEBUGGER_DATA64`, sadrži bitne reference poput `PsActiveProcessHead`. Ova specifična referenca pokazuje na glavu liste procesa, omogućavajući prikazivanje svih procesa, što je osnovno za temeljnu analizu memorije.

## Informacije o operativnom sistemu

```bash
#vol3 has a plugin to give OS information (note that imageinfo from vol2 will give you OS info)
./vol.py -f file.dmp windows.info.Info
```

Plugin `banners.Banners` može se koristiti u **vol3 da bi se pokušalo pronaći linux banere** u dumpu.

## Hesovi/Lozinke

Izvucite SAM hesove, [keširane kredencijale domena](../../../windows-hardening/stealing-credentials/credentials-protections.md#cached-credentials) i [lsa tajne](../../../windows-hardening/authentication-credentials-uac-and-efs/#lsa-secrets).

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.hashdump.Hashdump #Grab common windows hashes (SAM+SYSTEM)
./vol.py -f file.dmp windows.cachedump.Cachedump #Grab domain cache hashes inside the registry
./vol.py -f file.dmp windows.lsadump.Lsadump #Grab lsa secrets
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispisa u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **privs**: Prikazuje privilegije za određeni proces.
* **getsids**: Prikazuje SID-ove za određeni proces.
* **envars**: Prikazuje okruženje za određeni proces.
* **cmdscan**: Skenira memorijski ispisa u potrazi za komandama.
* **consoles**: Prikazuje listu konzola.
* **screenshots**: Pravi snimke ekrana.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.

**Dodatni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheat%20Sheet.pdf)

```bash
volatility --profile=Win7SP1x86_23418 hashdump -f file.dmp #Grab common windows hashes (SAM+SYSTEM)
volatility --profile=Win7SP1x86_23418 cachedump -f file.dmp #Grab domain cache hashes inside the registry
volatility --profile=Win7SP1x86_23418 lsadump -f file.dmp #Grab lsa secrets
```
{% endtab %}
{% endtabs %}

## Damp memorije

Damp memorije procesa će **izvući sve** trenutno stanje procesa. Modul **procdump** će samo **izvući** kod.

```
volatility -f file.dmp --profile=Win7SP1x86 memdump -p 2168 -D conhost/
```

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj u oblasti kibernetičke bezbednosti u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je vrelo mesto susreta tehnoloških i kibernetičkih stručnjaka u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

## Procesi

### Lista procesa

Pokušajte da pronađete **sumnjive** procese (po imenu) ili **neočekivane** podprocese (na primer cmd.exe kao podproces iexplorer.exe).\
Bilo bi interesantno **uporediti** rezultat pslist sa rezultatom psscan kako biste identifikovali skrivene procese.

```bash
python3 vol.py -f file.dmp windows.pstree.PsTree # Get processes tree (not hidden)
python3 vol.py -f file.dmp windows.pslist.PsList # Get process list (EPROCESS)
python3 vol.py -f file.dmp windows.psscan.PsScan # Get hidden process list(malware)
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **kpcrscan**: Skenira memorijski ispisa u potrazi za KPCR strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova.
* **handles**: Prikazuje listu otvorenih ručki.
* **cmdline**: Prikazuje argumente komandne linije za svaki proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispisa u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih datoteka.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **mbrparser**: Prikazuje Master Boot Record (MBR) informacije.
* **yarascan**: Skenira memorijski ispisa koristeći YARA pravila.
* **vadinfo**: Prikazuje informacije o Virtual Address Descriptor (VAD).
* **vaddump**: Izvlači sadržaj VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do VAD-a.

**Primeri korišćenja**

*   Prikaz informacija o memorijskom ispisa:

    ```
    volatility -f memory_dump.mem imageinfo
    ```
*   Prikaz liste procesa:

    ```
    volatility -f memory_dump.mem pslist
    ```
*   Prikaz stabla procesa:

    ```
    volatility -f memory_dump.mem pstree
    ```
*   Prikaz otvorenih fajlova:

    ```
    volatility -f memory_dump.mem filescan
    ```
*   Izvlačenje lozinki iz memorijskog ispisa:

    ```
    volatility -f memory_dump.mem hashdump
    ```
*   Skeniranje memorijskog ispisa koristeći YARA pravila:

    ```
    volatility -f memory_dump.mem yarascan -Y "yara_rules.yar"
    ```
*   Prikaz informacija o Virtual Address Descriptor (VAD):

    ```
    volatility -f memory_dump.mem vadinfo
    ```
*   Izvlačenje sadržaja VAD-a:

    ```
    volatility -f memory_dump.mem vaddump -D output_directory/ -p <PID>
    ```
*   Prikaz stabla VAD-a:

    ```
    volatility -f memory_dump.mem vadtree
    ```
*   Prikaz putanje do VAD-a:

    ```
    volatility -f memory_dump.mem vadwalk -p <PID>
    ```

**Dodatni resursi**

* [Zvanična dokumentacija Volatility-a](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-files/blob/master/volatility/Volatility\_Cheat\_Sheet\_v2.6.pdf)

```bash
volatility --profile=PROFILE pstree -f file.dmp # Get process tree (not hidden)
volatility --profile=PROFILE pslist -f file.dmp # Get process list (EPROCESS)
volatility --profile=PROFILE psscan -f file.dmp # Get hidden process list(malware)
volatility --profile=PROFILE psxview -f file.dmp # Get hidden process list
```

### Dump proc

{% tabs %}
{% tab title="vol3" %}
Koristite sledeću komandu da biste izvršili dump procesa:

```bash
volatility -f <dump_file> --profile=<profile> procdump -p <pid> -D <output_directory>
```

Gde su sledeći parametri:

* `<dump_file>`: Putanja do fajla sa dumpom memorije.
* `<profile>`: Profil za analizu dumpa memorije.
* `<pid>`: ID procesa koji želite da dumpujete.
* `<output_directory>`: Putanja do direktorijuma gde će biti smešteni dump fajlovi.

Na primer, ako želite da izvršite dump procesa sa ID-em 1234 iz dump fajla "memory.dmp" koristeći profil "Win7SP1x64", koristite sledeću komandu:

```bash
volatility -f memory.dmp --profile=Win7SP1x64 procdump -p 1234 -D /path/to/output_directory
```

Ova komanda će izvršiti dump procesa sa ID-em 1234 iz fajla "memory.dmp" i smeštati dump fajlove u direktorijum "/path/to/output\_directory".
{% endtab %}
{% endtabs %}

```bash
./vol.py -f file.dmp windows.dumpfiles.DumpFiles --pid <pid> #Dump the .exe and dlls of the process in the current directory
```

```bash
volatility --profile=Win7SP1x86_23418 procdump --pid=3152 -n --dump-dir=. -f file.dmp
```

### Komandna linija

Da li je izvršeno nešto sumnjivo?

{% tabs %}
{% tab title="vol3" %}
```bash
python3 vol.py -f file.dmp windows.cmdline.CmdLine #Display process command-line arguments
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisu.
* **kdbgscan**: Skenira memorijski ispis u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispis u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispis u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispis u potrazi za sumnjivim procesima.
* **dumpfiles**: Izvlači fajlove iz memorijskog ispisa.
* **hashdump**: Izvlači korisničke lozinke iz memorijskog ispisa.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hivedump**: Izvlači registarski ključ iz memorijskog ispisa.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.

**Korisni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-files/blob/master/volatility/Volatility%20Cheat%20Sheet.pdf)
* [Volatility plugini](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#plugins)

```bash
volatility --profile=PROFILE cmdline -f file.dmp #Display process command-line arguments
volatility --profile=PROFILE consoles -f file.dmp #command history by scanning for _CONSOLE_INFORMATION
```
{% endtab %}
{% endtabs %}

Komande izvršene u `cmd.exe` se upravljaju putem **`conhost.exe`** (ili `csrss.exe` na sistemima pre Windows 7). To znači da, ako je **`cmd.exe`** prekinut od strane napadača pre nego što je dobijena memorija, i dalje je moguće povratiti istoriju komandi sesije iz memorije **`conhost.exe`**. Da biste to uradili, ako se detektuje neobična aktivnost unutar modula konzole, treba da se izvrši dump memorije povezanog procesa **`conhost.exe`**. Zatim, pretraživanjem **stringova** unutar ovog dumpa, moguće je izvući komandne linije korištene u sesiji.

### Okruženje

Dobijte vrednosti okruženja za svaki pokrenuti proces. Mogu postojati neke interesantne vrednosti.

```bash
python3 vol.py -f file.dmp windows.envars.Envars [--pid <pid>] #Display process environment variables
```

## Osnovna forenzička metodologija

### Analiza memorijskog ispisa

#### Volatility Cheat Sheet

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

#### Instalacija Volatility-a

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

#### Osnovne komande

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **dumpfiles**: Izvlači otvorene fajlove iz memorijskog ispisa.
* **hashdump**: Izvlači korisničke lozinke iz memorijskog ispisa.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hivedump**: Izvlači registarski ključ iz memorijskog ispisa.
* **printkey**: Prikazuje sadržaj registarskog ključa.
* **printval**: Prikazuje vrednost registarskog ključa.
* **mftparser**: Analizira Master File Table (MFT) iz memorijskog ispisa.
* **usnparser**: Analizira Update Sequence Number (USN) iz memorijskog ispisa.

#### Napredne tehnike

* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.

#### Korisni resursi

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-files/blob/master/volatility/Volatility%20Cheat%20Sheet.pdf)
* [Volatility plugini](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)

```bash
volatility --profile=PROFILE envars -f file.dmp [--pid <pid>] #Display process environment variables

volatility --profile=PROFILE -f file.dmp linux_psenv [-p <pid>] #Get env of process. runlevel var means the runlevel where the proc is initated
```

### Token privilegije

Proverite privilegije tokena u neočekivanim uslugama.\
Bilo bi interesantno napraviti listu procesa koji koriste privilegovani token.

```bash
#Get enabled privileges of some processes
python3 vol.py -f file.dmp windows.privileges.Privs [--pid <pid>]
#Get all processes with interesting privileges
python3 vol.py -f file.dmp windows.privileges.Privs | grep "SeImpersonatePrivilege\|SeAssignPrimaryPrivilege\|SeTcbPrivilege\|SeBackupPrivilege\|SeRestorePrivilege\|SeCreateTokenPrivilege\|SeLoadDriverPrivilege\|SeTakeOwnershipPrivilege\|SeDebugPrivilege"
```

## Osnovna forenzička metodologija

### Analiza memorijskog ispisa

#### Volatility Cheat Sheet

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

#### Instalacija Volatility-a

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

#### Osnovne komande

* **imageinfo**: Prikazuje informacije o memorijskom ispisu.
* **kdbgscan**: Skenira memorijski ispis u potrazi za adresom debugera.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispis u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispis u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispis u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispis u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispis u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **mbrparser**: Prikazuje Master Boot Record (MBR).
* **yarascan**: Skenira memorijski ispis koristeći YARA pravila.
* **vadinfo**: Prikazuje informacije o Virtual Address Descriptor (VAD).
* **vaddump**: Izvlači sadržaj VAD-a iz memorijskog ispisa.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* \*\*v

```bash
#Get enabled privileges of some processes
volatility --profile=Win7SP1x86_23418 privs --pid=3152 -f file.dmp | grep Enabled
#Get all processes with interesting privileges
volatility --profile=Win7SP1x86_23418 privs -f file.dmp | grep "SeImpersonatePrivilege\|SeAssignPrimaryPrivilege\|SeTcbPrivilege\|SeBackupPrivilege\|SeRestorePrivilege\|SeCreateTokenPrivilege\|SeLoadDriverPrivilege\|SeTakeOwnershipPrivilege\|SeDebugPrivilege"
```

### SIDs

Proverite svaki SSID koji je u vlasništvu procesa.\
Bilo bi interesantno izlistati procese koji koriste privilegovanu SSID (i procese koji koriste neku servisnu SSID).

```bash
./vol.py -f file.dmp windows.getsids.GetSIDs [--pid <pid>] #Get SIDs of processes
./vol.py -f file.dmp windows.getservicesids.GetServiceSIDs #Get the SID of services
```

## Osnovna forenzička metodologija

### Analiza memorijskog ispisa

#### Volatility Cheat Sheet

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

#### Instalacija Volatility-a

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

#### Osnovne komande

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispisa u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **privs**: Prikazuje privilegije za određeni proces.
* **getsids**: Prikazuje SID-ove za određeni proces.
* **envars**: Prikazuje okruženje za određeni proces.
* **cmdscan**: Skenira memorijski ispisa u potrazi za komandama.
* **consoles**: Prikazuje listu konzola.
* **screenshots**: Pravi snimke ekrana.

#### Napredne tehnike

* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.

#### Dodatni resursi

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheat%20Sheet.pdf)

```bash
volatility --profile=Win7SP1x86_23418 getsids -f file.dmp #Get the SID owned by each process
volatility --profile=Win7SP1x86_23418 getservicesids -f file.dmp #Get the SID of each service
```

### Drške

Korisno je znati kojim drugim datotekama, ključevima, nitima, procesima... **proces ima dršku** (otvoreno).

{% tabs %}
{% tab title="vol3" %}
```bash
vol.py -f file.dmp windows.handles.Handles [--pid <pid>]
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **dumpfiles**: Izvlači otvorene fajlove iz memorijskog ispisa.
* **hashdump**: Izvlači korisničke lozinke iz memorijskog ispisa.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hivedump**: Izvlači registarski ključ iz memorijskog ispisa.
* **printkey**: Prikazuje sadržaj registarskog ključa.
* **printval**: Prikazuje vrednost registarskog ključa.
* **mftparser**: Analizira Master File Table (MFT) iz memorijskog ispisa.
* **usnparser**: Analizira Update Sequence Number (USN) iz memorijskog ispisa.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.

**Korisni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-files/blob/master/volatility/Volatility%20Cheat%20Sheet.pdf)
* [Volatility plugini](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#plugins)

```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp handles [--pid=<pid>]
```
{% endtab %}
{% endtabs %}

### DLL-ovi

{% tabs %}
{% tab title="undefined" %}
```bash
./vol.py -f file.dmp windows.dlllist.DllList [--pid <pid>] #List dlls used by each
./vol.py -f file.dmp windows.dumpfiles.DumpFiles --pid <pid> #Dump the .exe and dlls of the process in the current directory process
```
{% endtab %}

{% tab title="vol2" %}
```bash
volatility --profile=Win7SP1x86_23418 dlllist --pid=3152 -f file.dmp #Get dlls of a proc
volatility --profile=Win7SP1x86_23418 dlldump --pid=3152 --dump-dir=. -f file.dmp #Dump dlls of a proc
```
{% endtab %}
{% endtabs %}

### Strings per processes

Volatility nam omogućava da proverimo kojem procesu pripada određeni string.

{% tabs %}
{% tab title="vol3" %}
```bash
strings file.dmp > /tmp/strings.txt
./vol.py -f /tmp/file.dmp windows.strings.Strings --strings-file /tmp/strings.txt
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **kpcrscan**: Skenira memorijski ispisa u potrazi za KPCR strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova.
* **handles**: Prikazuje listu otvorenih ručki.
* **cmdline**: Prikazuje argumente komandne linije za svaki proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **gdt**: Prikazuje Global Descriptor Table (GDT).
* **idt**: Prikazuje Interrupt Descriptor Table (IDT).
* **callbacks**: Prikazuje listu callback funkcija.
* **driverirp**: Prikazuje IRP strukturu za drajvere.
* **devicetree**: Prikazuje stablo uređaja.
* **privs**: Prikazuje privilegije za svaki proces.
* **getsids**: Prikazuje SID-ove za svaki proces.
* **envars**: Prikazuje okruženje za svaki proces.
* **svcscan**: Skenira memorijski ispisa u potrazi za Windows servisima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **gdt**: Prikazuje Global Descriptor Table (GDT).
* **idt**: Prikazuje Interrupt Descriptor Table (IDT).
* **callbacks**: Prikazuje listu callback funkcija.
* **driverirp**: Prikazuje IRP strukturu za drajvere.
* **devicetree**: Prikazuje stablo uređaja.
* **privs**: Prikazuje privilegije za svaki proces.
* **getsids**: Prikazuje SID-ove za svaki proces.
* **envars**: Prikazuje okruženje za svaki proces.
* **svcscan**: Skenira memorijski ispisa u potrazi za Windows servisima.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.

**Korisni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheat%20Sheet.pdf)

**Reference**

* [https://www.volatilityfoundation.org/](https://www.volatilityfoundation.org/)
* [https://github.com/volatilityfoundation/volatility](https://github.com/volatilityfoundation/volatility)

```bash
strings file.dmp > /tmp/strings.txt
volatility -f /tmp/file.dmp windows.strings.Strings --string-file /tmp/strings.txt

volatility -f /tmp/file.dmp --profile=Win81U1x64 memdump -p 3532 --dump-dir .
strings 3532.dmp > strings_file
```
{% endtab %}
{% endtabs %}

Takođe omogućava pretragu stringova unutar procesa koristeći modul yarascan:

```bash
./vol.py -f file.dmp windows.vadyarascan.VadYaraScan --yara-rules "https://" --pid 3692 3840 3976 3312 3084 2784
./vol.py -f file.dmp yarascan.YaraScan --yara-rules "https://"
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisu.
* **kdbgscan**: Skenira memorijski ispis u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispis u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispis u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispis u potrazi za sumnjivim procesima.
* **dumpfiles**: Izvlači fajlove iz memorijskog ispisa.
* **hashdump**: Izvlači korisničke lozinke iz memorijskog ispisa.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hivedump**: Izvlači registarski ključ iz memorijskog ispisa.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.
* **malfind**: Pronalazi sumnjive procese i module u memorijskom ispisu.

**Dodatni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-files/blob/master/volatility/Volatility%20Cheat%20Sheet.pdf)

```bash
volatility --profile=Win7SP1x86_23418 yarascan -Y "https://" -p 3692,3840,3976,3312,3084,2784
```

### UserAssist

**Windows** beleži programe koje pokrećete koristeći funkciju u registru nazvanu **UserAssist ključevi**. Ovi ključevi beleže koliko puta je svaki program izvršen i kada je poslednji put pokrenut.

```bash
./vol.py -f file.dmp windows.registry.userassist.UserAssist
```

## Osnovna forenzička metodologija

### Analiza memorijskog ispisa

#### Volatility Cheat Sheet

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

#### Instalacija Volatility-a

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

#### Osnovne komande

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispisa u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **privs**: Prikazuje privilegije za određeni proces.
* **getsids**: Prikazuje SID-ove za određeni proces.
* **envars**: Prikazuje okruženje za određeni proces.
* **cmdscan**: Skenira memorijski ispisa u potrazi za komandama.
* **consoles**: Prikazuje listu konzola.
* **screenshots**: Pravi snimke ekrana.

#### Napredne tehnike

* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.

#### Dodatni resursi

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheat%20Sheet.pdf)

```
volatility --profile=Win7SP1x86_23418 -f file.dmp userassist
```

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj u oblasti kibernetičke bezbednosti u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je vrelo susretište za profesionalce iz oblasti tehnologije i kibernetičke bezbednosti u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

## Usluge

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.svcscan.SvcScan #List services
./vol.py -f file.dmp windows.getservicesids.GetServiceSIDs #Get the SID of services
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisu.
* **kdbgscan**: Skenira memorijski ispis u potrazi za adresom debugera.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispis u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispis u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispis u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispis u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispis u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **connections**: Prikazuje aktivne mrežne konekcije.
* **connscan**: Skenira memorijski ispis u potrazi za mrežnim konekcijama.
* **netscan**: Skenira memorijski ispis u potrazi za mrežnim artefaktima.
* **vadinfo**: Prikazuje informacije o Virtual Address Descriptor (VAD).
* **vaddump**: Dumpuje sadržaj VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **memdump**: Dumpuje sadržaj određenog memorijskog regiona.
* **memmap**: Prikazuje mapu memorijskog ispisa.
* **memstrings**: Prikazuje stringove u memorijskom ispisu.
* **memdump**: Dumpuje sadržaj određenog memorijskog regiona.
* **memmap**: Prikazuje mapu memorijskog ispisa.
* **memstrings**: Prikazuje stringove u memorijskom ispisu.

**Napredne tehnike**

* **timeliner**: Generiše vremensku liniju događaja na osnovu memorijskog ispisa.
* **mftparser**: Analizira Master File Table (MFT) za NTFS particiju.
* **usnparser**: Analizira Update Sequence Number (USN) journal za NTFS particiju.
* **shellbags**: Analizira ShellBags artefakte.
* **hivelist**: Prikazuje listu učitanih Windows registarskih datoteka.
* **hivedump**: Dumpuje sadržaj Windows registarske datoteke.
* **hashdump**: Dumpuje korisničke lozinke iz memorijskog ispisa.
* **lsadump**: Dumpuje korisničke lozinke iz Security Account Manager (SAM) baze podataka.
* **mimikatz**: Izvršava Mimikatz alat za izvlačenje lozinki iz memorijskog ispisa.
* **yarascan**: Skenira memorijski ispis koristeći YARA pravila.
* **yarascan**: Skenira memorijski ispis koristeći YARA pravila.
* **dumpregistry**: Dumpuje Windows registar iz memorijskog ispisa.
* **dumpregistry**: Dumpuje Windows registar iz memorijskog ispisa.
* **dumpfiles**: Dumpuje fajlove iz memorijskog ispisa.
* **dumpfiles**: Dumpuje fajlove iz memorijskog ispisa.
* **dumpcerts**: Dumpuje digitalne sertifikate iz memorijskog ispisa.
* **dumpcerts**: Dumpuje digitalne sertifikate iz memorijskog ispisa.
* **dumpcache**: Dumpuje keširane fajlove iz memorijskog ispisa.
* **dumpcache**: Dumpuje keširane fajlove iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**: Dumpuje sadržaj VAD-a iz memorijskog ispisa.
* **dumpvad**:

```bash
#Get services and binary path
volatility --profile=Win7SP1x86_23418 svcscan -f file.dmp
#Get name of the services and SID (slow)
volatility --profile=Win7SP1x86_23418 getservicesids -f file.dmp
```
{% endtab %}
{% endtabs %}

## Mreža

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.netscan.NetScan
#For network info of linux use volatility2
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **dumpfiles**: Izvlači otvorene fajlove iz memorijskog ispisa.
* **hashdump**: Izvlači korisničke lozinke iz memorijskog ispisa.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hivedump**: Izvlači registarski ključ iz memorijskog ispisa.
* **printkey**: Prikazuje sadržaj registarskog ključa.
* **printval**: Prikazuje vrednost registarskog ključa.
* **mftparser**: Analizira Master File Table (MFT) iz memorijskog ispisa.
* **usnparser**: Analizira Update Sequence Number (USN) iz memorijskog ispisa.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.

**Korisni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-files/blob/master/volatility/Volatility%20Cheat%20Sheet.pdf)
* [Volatility plugini](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)

```bash
volatility --profile=Win7SP1x86_23418 netscan -f file.dmp
volatility --profile=Win7SP1x86_23418 connections -f file.dmp#XP and 2003 only
volatility --profile=Win7SP1x86_23418 connscan -f file.dmp#TCP connections
volatility --profile=Win7SP1x86_23418 sockscan -f file.dmp#Open sockets
volatility --profile=Win7SP1x86_23418 sockets -f file.dmp#Scanner for tcp socket objects

volatility --profile=SomeLinux -f file.dmp linux_ifconfig
volatility --profile=SomeLinux -f file.dmp linux_netstat
volatility --profile=SomeLinux -f file.dmp linux_netfilter
volatility --profile=SomeLinux -f file.dmp linux_arp #ARP table
volatility --profile=SomeLinux -f file.dmp linux_list_raw #Processes using promiscuous raw sockets (comm between processes)
volatility --profile=SomeLinux -f file.dmp linux_route_cache
```
{% endtab %}
{% endtabs %}

## Registarski panj

### Ispis dostupnih panjeva

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.registry.hivelist.HiveList #List roots
./vol.py -f file.dmp windows.registry.printkey.PrintKey #List roots and get initial subkeys
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **dumpfiles**: Izvlači otvorene fajlove iz memorijskog ispisa.
* **hashdump**: Izvlači korisničke lozinke iz memorijskog ispisa.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hivedump**: Izvlači registarski ključ iz memorijskog ispisa.
* **printkey**: Prikazuje sadržaj registarskog ključa.
* **printval**: Prikazuje vrednost registarskog ključa.
* **mftparser**: Analizira Master File Table (MFT) iz memorijskog ispisa.
* **usnparser**: Analizira Update Sequence Number (USN) iz memorijskog ispisa.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.

**Korisni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-files/blob/master/volatility/Volatility%20Cheat%20Sheet.pdf)
* [Volatility plugini](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#plugins)

```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp hivelist #List roots
volatility --profile=Win7SP1x86_23418 -f file.dmp printkey #List roots and get initial subkeys
```
{% endtab %}
{% endtabs %}

### Dobijanje vrednosti

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.registry.printkey.PrintKey --key "Software\Microsoft\Windows NT\CurrentVersion"
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **dumpfiles**: Izvlači otvorene fajlove iz memorijskog ispisa.
* **hashdump**: Izvlači korisničke lozinke iz memorijskog ispisa.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hivedump**: Izvlači registarski ključ iz memorijskog ispisa.
* **printkey**: Prikazuje sadržaj registarskog ključa.
* **printval**: Prikazuje vrednost registarskog ključa.
* **mftparser**: Analizira Master File Table (MFT) iz memorijskog ispisa.
* **usnparser**: Analizira Update Sequence Number (USN) iz memorijskog ispisa.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.
* **malfind**: Pronalazi sumnjive procese i DLL-ove.

**Korisni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-files/blob/master/volatility/Volatility%20Cheat%20Sheet.pdf)
* [Volatility plugini](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)
* [Volatility plugini - dodatni](https://github.com/tribalchicken/volatility-plugins)

```bash
volatility --profile=Win7SP1x86_23418 printkey -K "Software\Microsoft\Windows NT\CurrentVersion" -f file.dmp
# Get Run binaries registry value
volatility -f file.dmp --profile=Win7SP1x86 printkey -o 0x9670e9d0 -K 'Software\Microsoft\Windows\CurrentVersion\Run'
```

{% tabs %}
{% tab title="Opis" %}
Ova metoda se koristi za izradu memorijskog ispisa (dump) procesa ili sistema. Memorijski ispis može biti koristan za analizu i pronalaženje tragova napada ili sumnjive aktivnosti.

**Komande**

* `volatility -f <dump_file> imageinfo` - Prikazuje informacije o memorijskom ispisu.
* `volatility -f <dump_file> --profile=<profile> pslist` - Prikazuje listu procesa.
* `volatility -f <dump_file> --profile=<profile> psscan` - Skenira memorijski ispis i prikazuje listu procesa.
* `volatility -f <dump_file> --profile=<profile> pstree` - Prikazuje stablo procesa.
* `volatility -f <dump_file> --profile=<profile> dlllist -p <pid>` - Prikazuje listu učitanih DLL-ova za određeni proces.
* `volatility -f <dump_file> --profile=<profile> handles -p <pid>` - Prikazuje listu otvorenih ručki za određeni proces.
* `volatility -f <dump_file> --profile=<profile> filescan` - Skenira memorijski ispis i prikazuje listu otvorenih fajlova.
* `volatility -f <dump_file> --profile=<profile> cmdline -p <pid>` - Prikazuje komandnu liniju za određeni proces.
* `volatility -f <dump_file> --profile=<profile> consoles` - Prikazuje listu konzola.
* `volatility -f <dump_file> --profile=<profile> netscan` - Skenira memorijski ispis i prikazuje listu aktivnih mrežnih konekcija.
* `volatility -f <dump_file> --profile=<profile> connections` - Prikazuje listu aktivnih mrežnih konekcija.
* `volatility -f <dump_file> --profile=<profile> sockscan` - Skenira memorijski ispis i prikazuje listu otvorenih soketa.
* `volatility -f <dump_file> --profile=<profile> malfind` - Skenira memorijski ispis i prikazuje potencijalno zlonamjerne procese.
* `volatility -f <dump_file> --profile=<profile> malprocfind` - Skenira memorijski ispis i prikazuje potencijalno zlonamjerne procese.
* `volatility -f <dump_file> --profile=<profile> maldrvfind` - Skenira memorijski ispis i prikazuje potencijalno zlonamjerne drajvere.
* `volatility -f <dump_file> --profile=<profile> modscan` - Skenira memorijski ispis i prikazuje listu učitanih modula.
* `volatility -f <dump_file> --profile=<profile> moddump -b <base_address> -D <output_directory>` - Izdvaja modul iz memorijskog ispisa.
* `volatility -f <dump_file> --profile=<profile> dumpfiles -Q <pid> -D <output_directory>` - Izdvaja fajlove otvorene od strane određenog procesa.
* `volatility -f <dump_file> --profile=<profile> dumpregistry -D <output_directory>` - Izdvaja Windows registar iz memorijskog ispisa.
* `volatility -f <dump_file> --profile=<profile> hivelist` - Prikazuje listu učitanih Windows registara.
* `volatility -f <dump_file> --profile=<profile> printkey -K <registry_key>` - Prikazuje sadržaj određenog Windows registarskog ključa.
* `volatility -f <dump_file> --profile=<profile> hashdump -y <registry_hive>` - Izdvaja korisničke lozinke iz Windows registra.
* `volatility -f <dump_file> --profile=<profile> userassist` - Prikazuje informacije o korisničkim aktivnostima.
* `volatility -f <dump_file> --profile=<profile> shimcache` - Prikazuje informacije o ShimCache-u.
* `volatility -f <dump_file> --profile=<profile> ldrmodules` - Prikazuje listu učitanih modula.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* `volatility -f <dump_file> --profile=<profile> getservicesids` - Prikazuje SID-ove servisa.
* `volatility -f <dump_file> --profile=<profile> getsids` - Prikazuje SID-ove korisnika.
* \`volatility -f \<dump\_file> --profile=

```bash
#Dump a hive
volatility --profile=Win7SP1x86_23418 hivedump -o 0x9aad6148 -f file.dmp #Offset extracted by hivelist
#Dump all hives
volatility --profile=Win7SP1x86_23418 hivedump -f file.dmp
```

### Fajl sistem

#### Montiranje

{% tabs %}
{% tab title="undefined" %}
```bash
#See vol2
```
{% endtab %}

{% tab title="vol2" %}
```bash
volatility --profile=SomeLinux -f file.dmp linux_mount
volatility --profile=SomeLinux -f file.dmp linux_recover_filesystem #Dump the entire filesystem (if possible)
```
{% endtab %}
{% endtabs %}

#### Skeniranje/dump

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.filescan.FileScan #Scan for files inside the dump
./vol.py -f file.dmp windows.dumpfiles.DumpFiles --physaddr <0xAAAAA> #Offset from previous command
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispisa u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **privs**: Prikazuje privilegije za određeni proces.
* **getsids**: Prikazuje SID-ove za određeni proces.
* **envars**: Prikazuje okruženje za određeni proces.
* **cmdscan**: Skenira memorijski ispisa u potrazi za komandama.
* **consoles**: Prikazuje listu konzola.
* **screenshots**: Pravi snimke ekrana.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.

**Dodatni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheat%20Sheet.pdf)

```bash
volatility --profile=Win7SP1x86_23418 filescan -f file.dmp #Scan for files inside the dump
volatility --profile=Win7SP1x86_23418 dumpfiles -n --dump-dir=/tmp -f file.dmp #Dump all files
volatility --profile=Win7SP1x86_23418 dumpfiles -n --dump-dir=/tmp -Q 0x000000007dcaa620 -f file.dmp

volatility --profile=SomeLinux -f file.dmp linux_enumerate_files
volatility --profile=SomeLinux -f file.dmp linux_find_file -F /path/to/file
volatility --profile=SomeLinux -f file.dmp linux_find_file -i 0xINODENUMBER -O /path/to/dump/file
```
{% endtab %}
{% endtabs %}

#### Master File Table

{% tabs %}
{% tab title="vol3" %}
Master File Table (MFT) je ključna struktura u NTFS fajl sistemu koja čuva informacije o svim fajlovima i direktorijumima na disku. Analiza MFT-a može pružiti korisne informacije o aktivnostima na sistemu, kao što su kreiranje, brisanje i modifikacija fajlova.

**Pregled MFT-a**

Da biste pregledali MFT, možete koristiti sledeću komandu:

```bash
volatility -f <dump_file> --profile=<profile> mftparser
```

Ova komanda će izlistati sve fajlove i direktorijume sa njihovim atributima, kao što su vreme kreiranja, vreme poslednje modifikacije i veličina fajla.

**Izvlačenje fajlova iz MFT-a**

Da biste izvukli fajl iz MFT-a, možete koristiti sledeću komandu:

```bash
volatility -f <dump_file> --profile=<profile> dumpfiles -Q <file_record_number> -D <output_directory>
```

Gde `<file_record_number>` predstavlja broj zapisa fajla u MFT-u, a `<output_directory>` je direktorijum u koji će fajl biti izvučen.

**Analiza MFT-a**

Analiza MFT-a može pružiti korisne informacije o aktivnostima na sistemu, kao što su:

* Identifikacija fajlova koji su bili obrisani
* Identifikacija fajlova koji su bili modifikovani
* Identifikacija fajlova koji su bili kreirani
* Identifikacija fajlova koji su bili preimenovani

Ove informacije mogu biti korisne u istrazi incidenta ili forenzičkoj analizi.
{% endtab %}
{% endtabs %}

```bash
# I couldn't find any plugin to extract this information in volatility3
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4. Instalirajte Volatility pomoću pip komande: `pip install volatility`.

**Osnovne komande**

* `volatility -f <file> imageinfo`: Prikazuje informacije o memorijskom ispisa.
* `volatility -f <file> --profile=<profile> <command>`: Izvršava određenu komandu na memorijskom ispisa koristeći određeni profil.

**Prikaz informacija o memorijskom ispisa**

* `imageinfo`: Prikazuje informacije o memorijskom ispisa kao što su operativni sistem, arhitektura, datum i vreme snimanja.

**Analiza procesa**

* `pslist`: Prikazuje listu svih procesa u memorijskom ispisa.
* `psscan`: Skenira memorijski ispisa i prikazuje listu svih procesa.
* `pstree`: Prikazuje hijerarhijsku strukturu procesa.
* `dlllist`: Prikazuje listu učitanih DLL-ova za svaki proces.
* `handles`: Prikazuje listu otvorenih ručki za svaki proces.
* `cmdline`: Prikazuje argumente komandne linije za svaki proces.

**Analiza datoteka**

* `filescan`: Skenira memorijski ispisa i prikazuje listu svih otvorenih datoteka.
* `malfind`: Skenira memorijski ispisa i prikazuje sumnjive procese i datoteke.
* `dumpfiles -Q <PID>`: Izdvaja sve otvorene datoteke za određeni proces.

**Analiza registra**

* `hivelist`: Prikazuje listu učitanih registarskih datoteka.
* `printkey -K <HivePath>`: Prikazuje sadržaj određenog registarskog ključa.

**Analiza mreže**

* `netscan`: Prikazuje listu aktivnih mrežnih veza.
* `connscan`: Prikazuje listu aktivnih TCP veza.

**Analiza korisnika**

* `hivescan`: Prikazuje listu učitanih korisničkih profila.
* `userassist`: Prikazuje informacije o korisničkim aktivnostima.

**Analiza servisa**

* `svcscan`: Prikazuje listu registrovanih servisa.
* `svcscan -s`: Prikazuje listu servisa sa detaljnim informacijama.

**Analiza drajvera**

* `driverirp`: Prikazuje listu IRP (I/O Request Packet) za svaki drajver.
* `drivermodule`: Prikazuje listu učitanih drajvera.

**Analiza heševa**

* `hashdump`: Prikazuje heševe lozinki korisnika.

**Analiza malvera**

* `malfind`: Prikazuje sumnjive procese i datoteke.
* `malfind -p <PID>`: Prikazuje sumnjive procese i datoteke za određeni proces.

**Analiza memorije**

* `memdump -p <PID> -D <output_directory>`: Izdvaja memorijski ispisa određenog procesa.

**Analiza događaja**

* `evtlogs`: Prikazuje listu događaja iz Windows Event Log-a.

**Analiza USB uređaja**

* `usbscan`: Prikazuje listu povezanih USB uređaja.

**Analiza datuma i vremena**

* `timeliner`: Prikazuje listu događaja sortiranih po vremenu.

**Analiza fizičke memorije**

* `imagecopy`: Kopira fizičku memoriju u datoteku.

**Analiza struktura podataka**

* `vadinfo`: Prikazuje informacije o Virtual Address Descriptor (VAD) strukturi.

**Analiza procesa u realnom vremenu**

* `procdump -p <PID> -D <output_directory>`: Izdvaja memorijski ispisa određenog procesa u realnom vremenu.

**Analiza drajvera u realnom vremenu**

* `moddump -m <module_name> -D <output_directory>`: Izdvaja memorijski ispisa određenog drajvera u realnom vremenu.

**Analiza mreže u realnom vremenu**

* `connscan -t`: Prikazuje listu aktivnih TCP veza u realnom vremenu.

**Analiza USB uređaja u realnom vremenu**

* `usbscan -t`: Prikazuje listu povezanih USB uređaja u realnom vremenu.

**Analiza drajvera u realnom vremenu**

* `driverirp -t`: Prikazuje listu IRP (I/O Request Packet) za svaki drajver u realnom vremenu.

**Analiza heševa u realnom vremenu**

* `hashdump -t`: Prikazuje heševe lozinki korisnika u realnom vremenu.

**Analiza malvera u realnom vremenu**

* `malfind -t`: Prikazuje sumnjive procese i datoteke u realnom vremenu.

**Analiza događaja u realnom vremenu**

* `evtlogs -t`: Prikazuje listu događaja iz Windows Event Log-a u realnom vremenu.

**Analiza datuma i vremena u realnom vremenu**

* `timeliner -t`: Prikazuje listu događaja sortiranih po vremenu u realnom vremenu.

**Analiza fizičke memorije u realnom vremenu**

* `imagecopy -t`: Kopira fizičku memoriju u datoteku u realnom vremenu.

**Analiza struktura podataka u realnom vremenu**

* `vadinfo -t`: Prikazuje informacije o Virtual Address Descriptor (VAD) strukturi u realnom vremenu.

**Analiza procesa u realnom vremenu**

* `procdump -p <PID> -D <output_directory> -t`: Izdvaja memorijski ispisa određenog procesa u realnom vremenu.

**Analiza drajvera u realnom vremenu**

* `moddump -m <module_name> -D <output_directory> -t`: Izdvaja memorijski ispisa određenog drajvera u realnom vremenu.

**Analiza mreže u realnom vremenu**

* `connscan -t`: Prikazuje listu aktivnih TCP veza u realnom vremenu.

**Analiza USB uređaja u realnom vremenu**

* `usbscan -t`: Prikazuje listu povezanih USB uređaja u realnom vremenu.

**Analiza drajvera u realnom vremenu**

* `driverirp -t`: Prikazuje listu IRP (I/O Request Packet) za svaki drajver u realnom vremenu.

**Analiza heševa u realnom vremenu**

* `hashdump -t`: Prikazuje heševe lozinki korisnika u realnom vremenu.

**Analiza malvera u realnom vremenu**

* `malfind -t`: Prikazuje sumnjive procese i datoteke u realnom vremenu.

**Analiza događaja u realnom vremenu**

* `evtlogs -t`: Prikazuje listu događaja iz Windows Event Log-a u realnom vremenu.

**Analiza datuma i vremena u realnom vremenu**

* `timeliner -t`: Prikazuje listu događaja sortiranih po vremenu u realnom vremenu.

**Analiza fizičke memorije u realnom vremenu**

* `imagecopy -t`: Kopira fizičku memoriju u datoteku u realnom vremenu.

**Analiza struktura podataka u realnom vremenu**

* `vadinfo -t`: Prikazuje informacije o Virtual Address Descriptor (VAD) strukturi u realnom vremenu.

```bash
volatility --profile=Win7SP1x86_23418 mftparser -f file.dmp
```
{% endtab %}
{% endtabs %}

**NTFS fajl sistem** koristi ključnu komponentu poznatu kao _master file table_ (MFT). Ova tabela uključuje barem jedan unos za svaki fajl na volumenu, uključujući i sam MFT. Važni detalji o svakom fajlu, kao što su **veličina, vremenske oznake, dozvole i stvarni podaci**, su enkapsulirani unutar unosa MFT-a ili u oblastima van MFT-a, ali na koje se referišu ovi unosi. Više detalja može se pronaći u [zvaničnoj dokumentaciji](https://docs.microsoft.com/en-us/windows/win32/fileio/master-file-table).

#### SSL Ključevi/Sertifikati

{% tabs %}
{% tab title="vol3" %}
```bash
#vol3 allows to search for certificates inside the registry
./vol.py -f file.dmp windows.registry.certificates.Certificates
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **dumpfiles**: Izvlači fajlove iz memorijskog ispisa.
* **hashdump**: Izvlači korisničke lozinke iz memorijskog ispisa.
* **netscan**: Skenira memorijski ispisa u potrazi za otvorenim mrežnim konekcijama.
* **connscan**: Skenira memorijski ispisa u potrazi za aktivnim mrežnim konekcijama.
* **apihooks**: Prikazuje listu API hook-ova.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za učitanim modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **gdt**: Prikazuje Global Descriptor Table (GDT).
* **idt**: Prikazuje Interrupt Descriptor Table (IDT).
* **callbacks**: Prikazuje listu callback funkcija.
* **driverirp**: Prikazuje listu IRP struktura za drajvere.
* **devicetree**: Prikazuje stablo uređaja.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **printkey**: Prikazuje sadržaj registarskog ključa.
* **dumpregistry**: Izvlači registar iz memorijskog ispisa.
* **svcscan**: Skenira memorijski ispisa u potrazi za Windows servisima.
* **privs**: Prikazuje privilegije za određeni proces.
* **envars**: Prikazuje okruženje za određeni proces.
* **cmdscan**: Skenira memorijski ispisa u potrazi za komandama koje su izvršene.
* **consoles**: Prikazuje listu otvorenih konzola.
* **screenshots**: Izvlači snimke ekrana iz memorijskog ispisa.
* **vadinfo**: Prikazuje informacije o Virtual Address Descriptor (VAD) strukturi.
* **vaddump**: Izvlači VAD regione iz memorijskog ispisa.
* **vadtree**: Prikazuje stablo VAD regiona.
* **vadwalk**: Prikazuje putanju do određenog VAD regiona.
* **memdump**: Izvlači memorijski region iz memorijskog ispisa.
* **memmap**: Prikazuje mapu memorijskog ispisa.
* **memstrings**: Prikazuje stringove iz memorijskog ispisa.
* **memscan**: Skenira memorijski ispisa u potrazi za određenim stringom.
* **yarascan**: Skenira memorijski ispisa koristeći YARA pravila.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* \*\*malf

```bash
#vol2 allos you to search and dump certificates from memory
#Interesting options for this modules are: --pid, --name, --ssl
volatility --profile=Win7SP1x86_23418 dumpcerts --dump-dir=. -f file.dmp
```
{% endtab %}
{% endtabs %}

### Malver

```bash
./vol.py -f file.dmp windows.malfind.Malfind [--dump] #Find hidden and injected code, [dump each suspicious section]
#Malfind will search for suspicious structures related to malware
./vol.py -f file.dmp windows.driverirp.DriverIrp #Driver IRP hook detection
./vol.py -f file.dmp windows.ssdt.SSDT #Check system call address from unexpected addresses

./vol.py -f file.dmp linux.check_afinfo.Check_afinfo #Verifies the operation function pointers of network protocols
./vol.py -f file.dmp linux.check_creds.Check_creds #Checks if any processes are sharing credential structures
./vol.py -f file.dmp linux.check_idt.Check_idt #Checks if the IDT has been altered
./vol.py -f file.dmp linux.check_syscall.Check_syscall #Check system call table for hooks
./vol.py -f file.dmp linux.check_modules.Check_modules #Compares module list to sysfs info, if available
./vol.py -f file.dmp linux.tty_check.tty_check #Checks tty devices for hooks
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **kpcrscan**: Skenira memorijski ispisa u potrazi za KPCR strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova.
* **handles**: Prikazuje listu otvorenih ručki.
* **cmdline**: Prikazuje argumente komandne linije za svaki proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispisa u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih datoteka.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **mbrparser**: Prikazuje Master Boot Record (MBR) informacije.
* **yarascan**: Skenira memorijski ispisa koristeći YARA pravila.
* **vadinfo**: Prikazuje informacije o Virtual Address Descriptor (VAD).
* **vaddump**: Izvlači sadržaj VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do VAD-a.

**Primeri korišćenja**

*   Prikaz informacija o memorijskom ispisa:

    ```
    volatility -f memory_dump.raw imageinfo
    ```
*   Prikaz liste procesa:

    ```
    volatility -f memory_dump.raw pslist
    ```
*   Prikaz stabla procesa:

    ```
    volatility -f memory_dump.raw pstree
    ```
*   Prikaz otvorenih ručki:

    ```
    volatility -f memory_dump.raw handles
    ```
*   Izvlačenje lozinki iz memorijskog ispisa:

    ```
    volatility -f memory_dump.raw hashdump
    ```
*   Skeniranje memorijskog ispisa koristeći YARA pravila:

    ```
    volatility -f memory_dump.raw yarascan -Y "yara_rules.yar"
    ```
*   Prikaz informacija o Virtual Address Descriptor (VAD):

    ```
    volatility -f memory_dump.raw vadinfo
    ```
*   Izvlačenje sadržaja VAD-a:

    ```
    volatility -f memory_dump.raw vaddump -D output_directory/ -p <PID>
    ```
*   Prikaz stabla VAD-a:

    ```
    volatility -f memory_dump.raw vadtree
    ```
*   Prikaz putanje do VAD-a:

    ```
    volatility -f memory_dump.raw vadwalk -p <PID>
    ```

**Dodatni resursi**

* [Zvanična dokumentacija Volatility-a](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-files/blob/master/volatility/Volatility\_Cheat\_Sheet\_v2.6.pdf)

```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp malfind [-D /tmp] #Find hidden and injected code [dump each suspicious section]
volatility --profile=Win7SP1x86_23418 -f file.dmp apihooks #Detect API hooks in process and kernel memory
volatility --profile=Win7SP1x86_23418 -f file.dmp driverirp #Driver IRP hook detection
volatility --profile=Win7SP1x86_23418 -f file.dmp ssdt #Check system call address from unexpected addresses

volatility --profile=SomeLinux -f file.dmp linux_check_afinfo
volatility --profile=SomeLinux -f file.dmp linux_check_creds
volatility --profile=SomeLinux -f file.dmp linux_check_fop
volatility --profile=SomeLinux -f file.dmp linux_check_idt
volatility --profile=SomeLinux -f file.dmp linux_check_syscall
volatility --profile=SomeLinux -f file.dmp linux_check_modules
volatility --profile=SomeLinux -f file.dmp linux_check_tty
volatility --profile=SomeLinux -f file.dmp linux_keyboard_notifiers #Keyloggers
```
{% endtab %}
{% endtabs %}

### Skeniranje sa yara

Koristite ovaj skript za preuzimanje i spajanje svih yara pravila za malver sa github-a: [https://gist.github.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9](https://gist.github.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9)\
Napravite direktorijum _**rules**_ i izvršite skriptu. Ovo će kreirati fajl nazvan _**malware\_rules.yar**_ koji sadrži sva yara pravila za malver.

```bash
wget https://gist.githubusercontent.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9/raw/4ec711d37f1b428b63bed1f786b26a0654aa2f31/malware_yara_rules.py
mkdir rules
python malware_yara_rules.py
#Only Windows
./vol.py -f file.dmp windows.vadyarascan.VadYaraScan --yara-file /tmp/malware_rules.yar
#All
./vol.py -f file.dmp yarascan.YaraScan --yara-file /tmp/malware_rules.yar
```

```bash
wget https://gist.githubusercontent.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9/raw/4ec711d37f1b428b63bed1f786b26a0654aa2f31/malware_yara_rules.py
mkdir rules
python malware_yara_rules.py
volatility --profile=Win7SP1x86_23418 yarascan -y malware_rules.yar -f ch2.dmp | grep "Rule:" | grep -v "Str_Win32" | sort | uniq
```

## MISC

### Spoljni dodaci

Ako želite da koristite spoljne dodatke, pobrinite se da su fascikle vezane za dodatke prvi parametar koji se koristi.

```bash
./vol.py --plugin-dirs "/tmp/plugins/" [...]
```

## Osnovna forenzička metodologija

### Analiza memorijskog ispisa

#### Volatility Cheat Sheet

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

#### Instalacija Volatility-a

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

#### Osnovne komande

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **kpcrscan**: Skenira memorijski ispisa u potrazi za KPCR strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova.
* **handles**: Prikazuje listu otvorenih ručki.
* **cmdline**: Prikazuje argumente komandne linije za svaki proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispisa u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih datoteka.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **mbrparser**: Prikazuje Master Boot Record (MBR) informacije.
* **yarascan**: Skenira memorijski ispisa koristeći YARA pravila.

#### Primeri korišćenja

*   Prikaz informacija o memorijskom ispisa:

    ```
    volatility -f memory_dump.mem imageinfo
    ```
*   Prikaz liste procesa:

    ```
    volatility -f memory_dump.mem pslist
    ```
*   Prikaz stabla procesa:

    ```
    volatility -f memory_dump.mem pstree
    ```
*   Prikaz otvorenih ručki:

    ```
    volatility -f memory_dump.mem handles
    ```
*   Izvlačenje lozinki iz memorijskog ispisa:

    ```
    volatility -f memory_dump.mem hashdump
    ```
*   Skeniranje memorijskog ispisa koristeći YARA pravila:

    ```
    volatility -f memory_dump.mem yarascan -Y "yara_rules.yar"
    ```

#### Dodatni resursi

* [Zvanična dokumentacija Volatility-a](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheat%20Sheet.pdf)

```bash
volatilitye --plugins="/tmp/plugins/" [...]
```

#### Autoruns

Preuzmite ga sa [https://github.com/tomchop/volatility-autoruns](https://github.com/tomchop/volatility-autoruns)

```
volatility --plugins=volatility-autoruns/ --profile=WinXPSP2x86 -f file.dmp autoruns
```

### Mutexi

{% tabs %}
{% tab title="vol3" %}
```
./vol.py -f file.dmp windows.mutantscan.MutantScan
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4. Instalirajte Volatility pomoću pip komande: `pip install volatility`.

**Osnovne komande**

* `volatility -f <file> imageinfo`: Prikazuje informacije o memorijskom ispisa.
* `volatility -f <file> --profile=<profile> <command>`: Izvršava određenu komandu na memorijskom ispisa koristeći određeni profil.

**Prikaz informacija o memorijskom ispisa**

* `imageinfo`: Prikazuje informacije o memorijskom ispisa kao što su operativni sistem, arhitektura, verzija itd.

**Analiza procesa**

* `pslist`: Prikazuje listu svih procesa u memorijskom ispisa.
* `psscan`: Skenira memorijski ispisa i prikazuje informacije o svim procesima.
* `pstree`: Prikazuje hijerarhijski prikaz procesa u memorijskom ispisa.
* `dlllist`: Prikazuje listu učitanih DLL-ova za svaki proces.

**Analiza datoteka**

* `filescan`: Skenira memorijski ispisa i prikazuje informacije o svim otvorenim datotekama.
* `handles`: Prikazuje listu otvorenih ručki za svaki proces.
* `dumpfiles -Q <address>`: Izvlači datoteku iz memorijskog ispisa na određenoj adresi.

**Analiza registra**

* `hivelist`: Prikazuje listu učitanih registarskih ključeva.
* `printkey -K <address>`: Prikazuje sadržaj registarskog ključa na određenoj adresi.

**Analiza mreže**

* `connections`: Prikazuje listu aktivnih mrežnih veza.
* `connscan`: Skenira memorijski ispisa i prikazuje informacije o svim mrežnim vezama.

**Analiza korisnika**

* `hivescan`: Skenira memorijski ispisa i prikazuje informacije o svim učitanim korisničkim profilima.
* `hashdump -y <profile>`: Izvlači lozinke korisnika iz memorijskog ispisa.

**Analiza servisa**

* `svcscan`: Prikazuje listu svih servisa u memorijskom ispisa.
* `privs`: Prikazuje privilegije za svaki proces.

**Analiza drajvera**

* `driverirp`: Prikazuje listu IRP (I/O Request Packet) za svaki drajver.
* `drivermodule`: Prikazuje listu učitanih drajvera.

**Analiza rootkit-a**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa rootkit-om.
* `ssdt`: Prikazuje System Service Descriptor Table (SSDT) koja sadrži adrese sistemskih poziva.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* `eventhooks`: Prikazuje listu hook-ova događaja.

**Analiza memorije**

* `memdump -p <pid> -D <output_directory>`: Izvlači memorijski ispisa za određeni proces.
* `memmap`: Prikazuje mapu memorijskog ispisa.

**Analiza heuristika**

* `malfind`: Prikazuje sumnjive procese koji mogu biti povezani sa malverom.
* `ldrmodules`: Prikazuje listu učitanih modula za svaki proces.

**Analiza heševa**

* `hashdump`: Izvlači lozinke korisnika iz memorijskog ispisa.
* `hivelist`: Prikazuje listu učitanih registarskih ključeva.

**Analiza događaja**

* `evtlogs`: Prikazuje listu svih događaja u memorijskom ispisa.
* \`eventhooks

```bash
volatility --profile=Win7SP1x86_23418 mutantscan -f file.dmp
volatility --profile=Win7SP1x86_23418 -f file.dmp handles -p <PID> -t mutant
```
{% endtab %}
{% endtabs %}

### Simboličke veze

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.symlinkscan.SymlinkScan
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **kpcrscan**: Skenira memorijski ispisa u potrazi za KPCR strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova.
* **handles**: Prikazuje listu otvorenih ručki.
* **cmdline**: Prikazuje argumente komandne linije za svaki proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispisa u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih datoteka.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **mbrparser**: Prikazuje Master Boot Record (MBR) informacije.
* **yarascan**: Skenira memorijski ispisa koristeći YARA pravila.
* **vadinfo**: Prikazuje informacije o Virtual Address Descriptor (VAD).
* **vaddump**: Izvlači sadržaj VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do VAD-a.

**Napredne tehnike**

* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.
* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.
* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.
* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.
* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.
* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.
* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.
* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.
* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.
* **malfind**: Pronalazi sumnjive procese i modifikovane DLL-ove.

**Dodatni resursi**

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility GitHub repozitorijum](https://github.com/volatilityfoundation/volatility)

```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp symlinkscan
```
{% endtab %}
{% endtabs %}

### Bash

Moguće je **čitati iz memorije istoriju bash-a**. Takođe možete izvući datoteku _.bash\_history_, ali ako je onemogućena, bićete zadovoljni što možete koristiti ovaj modul volatilnosti.

```
./vol.py -f file.dmp linux.bash.Bash
```

## Osnovna forenzička metodologija

### Analiza memorijskog ispisa

#### Volatility Cheat Sheet

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

#### Instalacija Volatility-a

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

#### Osnovne komande

* **imageinfo**: Prikazuje informacije o memorijskom ispisa.
* **kdbgscan**: Skenira memorijski ispisa u potrazi za KDBG strukturom.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispisa u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispisa u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispisa u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispisa u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispisa u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **privs**: Prikazuje privilegije za određeni proces.
* **getsids**: Prikazuje SID-ove za određeni proces.
* **envars**: Prikazuje okruženje za određeni proces.
* **cmdscan**: Skenira memorijski ispisa u potrazi za komandama.
* **consoles**: Prikazuje listu konzola.
* **screenshots**: Pravi snimke ekrana.

#### Napredne tehnike

* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.
* **malfind**: Pronalazi sumnjive procese i modifikovane funkcije.

#### Dodatni resursi

* [Volatility dokumentacija](https://github.com/volatilityfoundation/volatility/wiki)
* [Volatility Cheat Sheet](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheat%20Sheet.pdf)

```
volatility --profile=Win7SP1x86_23418 -f file.dmp linux_bash
```

### Vremenska linija

{% tabs %}
{% tab title="undefined" %}
```bash
./vol.py -f file.dmp timeLiner.TimeLiner
```
{% endtab %}

{% tab title="vol2" %}
```
volatility --profile=Win7SP1x86_23418 -f timeliner
```
{% endtab %}
{% endtabs %}

### Drajveri

{% tabs %}
{% tab title="vol3" %}
```
./vol.py -f file.dmp windows.driverscan.DriverScan
```

### Osnovna forenzička metodologija

#### Analiza memorijskog ispisa

**Volatility Cheat Sheet**

Ovaj cheat sheet pruža pregled osnovnih komandi i tehnika koje se koriste u analizi memorijskog ispisa pomoću alata Volatility.

**Instalacija Volatility-a**

1. Preuzmite Volatility sa [zvanične stranice](https://www.volatilityfoundation.org/releases) i raspakujte ga.
2. Instalirajte Python 2.7.x.
3. Instalirajte pip.
4.  Instalirajte Volatility koristeći pip:

    ```
    pip install volatility
    ```

**Osnovne komande**

* **imageinfo**: Prikazuje informacije o memorijskom ispisu.
* **kdbgscan**: Skenira memorijski ispis u potrazi za adresom debugera.
* **pslist**: Prikazuje listu procesa.
* **pstree**: Prikazuje stablo procesa.
* **psscan**: Skenira memorijski ispis u potrazi za procesima.
* **dlllist**: Prikazuje listu učitanih DLL-ova za određeni proces.
* **handles**: Prikazuje listu otvorenih ručki za određeni proces.
* **cmdline**: Prikazuje argumente komandne linije za određeni proces.
* **filescan**: Skenira memorijski ispis u potrazi za otvorenim fajlovima.
* **malfind**: Skenira memorijski ispis u potrazi za sumnjivim procesima.
* **apihooks**: Prikazuje API hook-ove.
* **ldrmodules**: Prikazuje listu učitanih modula.
* **modscan**: Skenira memorijski ispis u potrazi za modulima.
* **ssdt**: Prikazuje System Service Descriptor Table (SSDT).
* **driverscan**: Skenira memorijski ispis u potrazi za drajverima.
* **devicetree**: Prikazuje stablo uređaja.
* **registry**: Prikazuje informacije o registru.
* **hivelist**: Prikazuje listu učitanih registarskih ključeva.
* **hashdump**: Izvlači lozinke iz memorijskog ispisa.
* **mbrparser**: Prikazuje Master Boot Record (MBR).
* **yarascan**: Skenira memorijski ispis koristeći YARA pravila.
* **vadinfo**: Prikazuje informacije o Virtual Address Descriptor (VAD).
* **vaddump**: Izvlači sadržaj VAD-a iz memorijskog ispisa.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* **vadwalk**: Prikazuje putanju do određenog VAD-a.
* **vadtree**: Prikazuje stablo VAD-a.
* \*\*v

```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp driverscan
```
{% endtab %}
{% endtabs %}

### Dobijanje sadržaja iz privremene memorije (clipboard)

```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 clipboard -f file.dmp
```

### Dobijanje istorije pretraživača Internet Explorer

Koristite sledeću komandu da biste dobili istoriju pretraživača Internet Explorer:

```bash
volatility -f <memory_dump> --profile=<profile> iehistory
```

Zamenite `<memory_dump>` sa putanjom do memorijskog dumpa i `<profile>` sa odgovarajućim profilom za analizu.

```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 iehistory -f file.dmp
```

### Dobijanje teksta iz beležnice

Koristite sledeću komandu da biste dobili tekst iz beležnice:

```bash
volatility -f memory_dump.vmem --profile=PROFILE notepad
```

Gde `memory_dump.vmem` predstavlja ime fajla sa memorijskim dumpom, a `PROFILE` predstavlja profil operativnog sistema.

```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 notepad -f file.dmp
```

### Снимак екрана

```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 screenshot -f file.dmp
```

### Master Boot Record (MBR)

### Master Boot Record (MBR) (Master Boot Rekord)

The Master Boot Record (MBR) is the first sector of a storage device (such as a hard disk) that contains the boot loader and partition table. It plays a crucial role in the boot process of a computer.

Master Boot Record (MBR) je prvi sektor uređaja za skladištenje (kao što je hard disk) koji sadrži boot loader i tabelu particija. Ima ključnu ulogu u procesu pokretanja računara.

### Volatility Commands for MBR Analysis

### Volatility Commands for MBR Analysis (Volatility komande za analizu MBR-a)

To analyze the Master Boot Record (MBR) using Volatility, you can use the following commands:

Za analizu Master Boot Record (MBR) pomoću Volatility-a, možete koristiti sledeće komande:

```
volatility -f <memory_dump> mbrparser
```

This command will parse the memory dump and extract information about the Master Boot Record (MBR).

Ova komanda će parsirati memory dump i izvući informacije o Master Boot Record (MBR).

```
volatility -f <memory_dump> mbrparser --output=html --output-file=<output_file>
```

This command will parse the memory dump and generate an HTML report with information about the Master Boot Record (MBR).

Ova komanda će parsirati memory dump i generisati HTML izveštaj sa informacijama o Master Boot Record (MBR).

### MBR Analysis Techniques

### Tehnike analize MBR-a

When analyzing the Master Boot Record (MBR), you can use various techniques to gather information and identify any malicious activity. Some common techniques include:

Prilikom analize Master Boot Record (MBR), možete koristiti različite tehnike za prikupljanje informacija i identifikaciju bilo kakve zlonamerne aktivnosti. Neke uobičajene tehnike uključuju:

* **Static Analysis**: This involves examining the binary code of the MBR to identify any suspicious or malicious instructions.
* **Statička analiza**: Ovo uključuje pregledanje binarnog koda MBR-a radi identifikacije sumnjivih ili zlonamernih instrukcija.
* **Dynamic Analysis**: This involves executing the MBR in a controlled environment (such as a virtual machine) to observe its behavior and identify any malicious actions.
* **Dinamička analiza**: Ovo uključuje izvršavanje MBR-a u kontrolisanom okruženju (kao što je virtuelna mašina) radi posmatranja njegovog ponašanja i identifikacije bilo kakvih zlonamernih radnji.
* **Signature-based Analysis**: This involves comparing the MBR against known signatures of malware to identify any matches.
* **Analiza na osnovu potpisa**: Ovo uključuje upoređivanje MBR-a sa poznatim potpisima zlonamernog softvera radi identifikacije podudaranja.
* **Behavioral Analysis**: This involves analyzing the behavior of the MBR during the boot process to identify any abnormal or suspicious activities.
* **Ponašajna analiza**: Ovo uključuje analizu ponašanja MBR-a tokom procesa pokretanja radi identifikacije bilo kakvih abnormalnih ili sumnjivih aktivnosti.

By using these techniques, you can gain valuable insights into the Master Boot Record (MBR) and detect any potential security threats.

Korišćenjem ovih tehnika, možete dobiti vredne uvide u Master Boot Record (MBR) i otkriti potencijalne sigurnosne pretnje.

```bash
volatility --profile=Win7SP1x86_23418 mbrparser -f file.dmp
```

**Master Boot Record (MBR)** ima ključnu ulogu u upravljanju logičkim particijama skladišnog medija, koje su strukturirane s različitim [datotečnim sustavima](https://en.wikipedia.org/wiki/File\_system). Ne samo da sadrži informacije o rasporedu particija, već također sadrži izvršni kod koji djeluje kao pokretač sustava za pokretanje. Taj pokretač sustava za pokretanje ili izravno pokreće postupak učitavanja drugog stupnja operativnog sustava (vidi [pokretač sustava za pokretanje drugog stupnja](https://en.wikipedia.org/wiki/Second-stage\_boot\_loader)) ili radi u skladu s [zapisom za pokretanje volumena](https://en.wikipedia.org/wiki/Volume\_boot\_record) (VBR) svake particije. Za dubinsko znanje, pogledajte [MBR stranicu na Wikipediji](https://en.wikipedia.org/wiki/Master\_boot\_record).

## Reference

* [https://andreafortuna.org/2017/06/25/volatility-my-own-cheatsheet-part-1-image-identification/](https://andreafortuna.org/2017/06/25/volatility-my-own-cheatsheet-part-1-image-identification/)
* [https://scudette.blogspot.com/2012/11/finding-kernel-debugger-block.html](https://scudette.blogspot.com/2012/11/finding-kernel-debugger-block.html)
* [https://or10nlabs.tech/cgi-sys/suspendedpage.cgi](https://or10nlabs.tech/cgi-sys/suspendedpage.cgi)
* [https://www.aldeid.com/wiki/Windows-userassist-keys](https://www.aldeid.com/wiki/Windows-userassist-keys) ​\* [https://learn.microsoft.com/en-us/windows/win32/fileio/master-file-table](https://learn.microsoft.com/en-us/windows/win32/fileio/master-file-table)
* [https://answers.microsoft.com/en-us/windows/forum/all/uefi-based-pc-protective-mbr-what-is-it/0fc7b558-d8d4-4a7d-bae2-395455bb19aa](https://answers.microsoft.com/en-us/windows/forum/all/uefi-based-pc-protective-mbr-what-is-it/0fc7b558-d8d4-4a7d-bae2-395455bb19aa)

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj o kibernetičkoj sigurnosti u **Španiji** i jedan od najvažnijih u **Europi**. S misijom promicanja tehničkog znanja, ovaj kongres je vruća točka susreta za stručnjake za tehnologiju i kibernetičku sigurnost u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>Naučite hakiranje AWS-a od nule do heroja s</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricksu:

* Ako želite vidjeti **oglašavanje vaše tvrtke u HackTricksu** ili **preuzeti HackTricks u PDF formatu**, provjerite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**službenu PEASS & HackTricks opremu**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podijelite svoje hakirajuće trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorije.

</details>
