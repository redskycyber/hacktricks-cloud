# Kerberos Double Hop Problem

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Uvod

Problem "dvostrukog skoka" u Kerberosu se javlja kada napadač pokušava da koristi **Kerberos autentifikaciju preko dva** **skoka**, na primer koristeći **PowerShell**/**WinRM**.

Kada se **autentifikacija** vrši putem **Kerberosa**, **poverilački podaci** se **ne čuvaju** u **memoriji**. Zato, ako pokrenete mimikatz, **nećete pronaći poverilačke podatke** korisnika na mašini, čak i ako on pokreće procese.

To je zato što kada se povezujete sa Kerberosom, sledeći su koraci:

1. Korisnik1 pruža poverilačke podatke i **kontroler domena** vraća Kerberos **TGT** korisniku1.
2. Korisnik1 koristi **TGT** da zatraži **servisni tiket** za **povezivanje** sa Serverom1.
3. Korisnik1 se **povezuje** sa **Serverom1** i pruža **servisni tiket**.
4. **Server1** **nema** poverilačke podatke korisnika1 u kešu niti **TGT** korisnika1. Zato, kada korisnik1 sa Servera1 pokuša da se prijavi na drugi server, on **ne može da se autentifikuje**.

### Neograničeno preusmeravanje

Ako je **neograničeno preusmeravanje** omogućeno na računaru, ovo se neće dogoditi jer će **Server** dobiti **TGT** svakog korisnika koji mu pristupa. Osim toga, ako se koristi neograničeno preusmeravanje, verovatno možete **ugroziti kontroler domena** iz njega.\
[**Više informacija na stranici o neograničenom preusmeravanju**](unconstrained-delegation.md).

### CredSSP

Još jedan način da se izbegne ovaj problem koji je [**posebno nesiguran**](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/enable-wsmancredssp?view=powershell-7) je **Credential Security Support Provider**. Prema Microsoft-u:

> CredSSP autentifikacija delegira korisničke poverilačke podatke sa lokalnog računara na udaljeni računar. Ova praksa povećava sigurnosni rizik udaljene operacije. Ako je udaljeni računar kompromitovan, kada mu se proslede poverilački podaci, poverilački podaci mogu se koristiti za kontrolu mrežne sesije.

Visoko se preporučuje da se **CredSSP** onemogući na proizvodnim sistemima, osetljivim mrežama i sličnim okruženjima zbog sigurnosnih razloga. Da biste utvrdili da li je **CredSSP** omogućen, može se pokrenuti komanda `Get-WSManCredSSP`. Ova komanda omogućava **proveru statusa CredSSP** i može se čak izvršiti i udaljeno, pod uslovom da je omogućen **WinRM**.

```powershell
Invoke-Command -ComputerName bizintel -Credential ta\redsuit -ScriptBlock {
Get-WSManCredSSP
}
```

## Rešenja

### Invoke Command

Da biste rešili problem dvostrukog skoka, predstavljen je metod koji uključuje ugnježđeni `Invoke-Command`. Ovo ne rešava problem direktno, već nudi alternativno rešenje bez potrebe za posebnim konfiguracijama. Pristup omogućava izvršavanje komande (`hostname`) na sekundarnom serveru putem PowerShell komande izvršene sa početnog napadačkog računara ili putem prethodno uspostavljene PS-Session sa prvom serverom. Evo kako se to radi:

```powershell
$cred = Get-Credential ta\redsuit
Invoke-Command -ComputerName bizintel -Credential $cred -ScriptBlock {
Invoke-Command -ComputerName secdev -Credential $cred -ScriptBlock {hostname}
}
```

Alternativno, predlaže se uspostavljanje PS-sesije sa prvom serverom i pokretanje `Invoke-Command` koristeći `$cred` radi centralizacije zadataka.

### Registrovanje PSSession konfiguracije

Rešenje za zaobilaženje problema dvostrukog skoka uključuje korišćenje `Register-PSSessionConfiguration` sa `Enter-PSSession`. Ovaj metod zahteva drugačiji pristup od `evil-winrm` i omogućava sesiju koja ne pati od ograničenja dvostrukog skoka.

```powershell
Register-PSSessionConfiguration -Name doublehopsess -RunAsCredential domain_name\username
Restart-Service WinRM
Enter-PSSession -ConfigurationName doublehopsess -ComputerName <pc_name> -Credential domain_name\username
klist
```

### PortForwarding

Za lokalne administratore na posrednom cilju, port forwarding omogućava slanje zahteva ka krajnjem serveru. Korišćenjem `netsh` komande, može se dodati pravilo za port forwarding, zajedno sa Windows firewall pravilom koje dozvoljava prosleđivanje porta.

```bash
netsh interface portproxy add v4tov4 listenport=5446 listenaddress=10.35.8.17 connectport=5985 connectaddress=10.35.8.23
netsh advfirewall firewall add rule name=fwd dir=in action=allow protocol=TCP localport=5446
```

#### winrs.exe

`winrs.exe` može se koristiti za prosleđivanje WinRM zahteva, potencijalno kao manje detektabilna opcija ako je praćenje PowerShell-a zabrinjavajuće. Sledeća komanda demonstrira njegovu upotrebu:

```bash
winrs -r:http://bizintel:5446 -u:ta\redsuit -p:2600leet hostname
```

### OpenSSH

Instaliranje OpenSSH-a na prvom serveru omogućava rešenje za problem dvostrukog preskakanja, posebno korisno u scenarijima sa skočnim kutijama. Ovaj metod zahteva CLI instalaciju i podešavanje OpenSSH-a za Windows. Kada je konfigurisan za autentifikaciju lozinkom, ovo omogućava posrednom serveru da dobije TGT u ime korisnika.

#### Koraci za instalaciju OpenSSH-a

1. Preuzmite najnoviji zip fajl sa OpenSSH izdanjem i premestite ga na ciljni server.
2. Raspakujte zip fajl i pokrenite skriptu `Install-sshd.ps1`.
3. Dodajte pravilo za otvaranje porta 22 na firewall-u i proverite da li SSH servisi rade.

Da biste rešili greške `Connection reset`, možda će biti potrebno ažurirati dozvole kako bi svi imali pristup čitanju i izvršavanju u OpenSSH direktorijumu.

```bash
icacls.exe "C:\Users\redsuit\Documents\ssh\OpenSSH-Win64" /grant Everyone:RX /T
```

## Reference

* [https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/razumevanje-kerberos-dvostrukog-skok/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/razumevanje-kerberos-dvostrukog-skok/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20)
* [https://posts.slayerlabs.com/double-hop/](https://posts.slayerlabs.com/double-hop/)
* [https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/još-jedno-rešenje-za-višestruko-prebacivanje-powershell-udaljavanja](https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/jo%C5%A1-jedno-re%C5%A1enje-za-vi%C5%A1estruko-prebacivanje-powershell-udaljavanja)
* [https://4sysops.com/archives/rešite-problem-višestrukog-prebacivanja-powershell-a-bez-korišćenja-credssp/](https://4sysops.com/archives/re%C5%A1ite-problem-vi%C5%A1estrukog-prebacivanja-powershell-a-bez-kori%C5%A1%C4%87enja-credssp/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
