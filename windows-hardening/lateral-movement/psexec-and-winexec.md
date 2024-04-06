# PsExec/Winexec/ScExec

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Kako funkcionišu

Proces je prikazan u sledećim koracima, ilustrujući kako se manipuliše binarnim fajlovima servisa kako bi se postiglo daljinsko izvršavanje na ciljnom računaru putem SMB-a:

1. **Kopiranje binarnog fajla servisa na ADMIN$ deljenje preko SMB-a** se vrši.
2. **Kreiranje servisa na udaljenom računaru** se vrši tako što se pokazuje na binarni fajl.
3. Servis se **pokreće udaljeno**.
4. Nakon završetka, servis se **zaustavlja, a binarni fajl se briše**.

### **Proces ručnog izvršavanja PsExec-a**

Pretpostavljajući da postoji izvršni payload (kreiran sa msfvenom i obfuskiran korišćenjem Veil-a kako bi se izbeglo otkrivanje antivirusnim programima), nazvan 'met8888.exe', koji predstavlja meterpreter reverse\_http payload, preduzimaju se sledeći koraci:

* **Kopiranje binarnog fajla**: Izvršni fajl se kopira na ADMIN$ deljenje iz komandne linije, iako se može postaviti bilo gde na fajl sistem kako bi ostao sakriven.
* **Kreiranje servisa**: Korišćenjem Windows `sc` komande, koja omogućava upitivanje, kreiranje i brisanje Windows servisa udaljeno, kreira se servis nazvan "meterpreter" koji pokazuje na preneti binarni fajl.
* **Pokretanje servisa**: Poslednji korak podrazumeva pokretanje servisa, što će verovatno rezultirati "time-out" greškom zbog toga što binarni fajl nije pravi binarni fajl servisa i ne vraća očekivani kod odgovora. Ova greška nije bitna jer je primarni cilj izvršavanje binarnog fajla.

Posmatranje Metasploit slušaoca će otkriti da je sesija uspešno pokrenuta.

[Saznajte više o `sc` komandi](https://technet.microsoft.com/en-us/library/bb490995.aspx).

Pronađite detaljnije korake na: [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

**Takođe možete koristiti Windows Sysinternals binarni fajl PsExec.exe:**

![](<../../.gitbook/assets/image (165).png>)

Takođe možete koristiti [**SharpLateral**](https://github.com/mertdas/SharpLateral):

{% code overflow="wrap" %}
```
SharpLateral.exe redexec HOSTNAME C:\\Users\\Administrator\\Desktop\\malware.exe.exe malware.exe ServiceName
```
{% endcode %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
