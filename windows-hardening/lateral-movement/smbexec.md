# SmbExec/ScExec

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Kako radi

**Smbexec** je alat koji se koristi za izvršavanje komandi na daljinu na Windows sistemima, sličan **Psexec**-u, ali izbegava postavljanje zlonamernih fajlova na ciljni sistem.

### Ključne tačke o **SMBExec**

* Radi tako što kreira privremenu uslugu (na primer, "BTOBTO") na ciljnom računaru kako bi izvršio komande putem cmd.exe (%COMSPEC%), bez ispuštanja bilo kakvih binarnih fajlova.
* Uprkos svojoj prikrivenoj metodi, generiše evidenciju događaja za svaku izvršenu komandu, pružajući oblik neinteraktivne "ljuske".
* Komanda za povezivanje korišćenjem **Smbexec**-a izgleda ovako:

```bash
smbexec.py WORKGROUP/genericuser:genericpassword@10.10.10.10
```

### Izvršavanje komandi bez binarnih datoteka

* **Smbexec** omogućava direktno izvršavanje komandi putem binarnih putanja usluga, eliminirajući potrebu za fizičkim binarnim datotekama na cilju.
* Ova metoda je korisna za izvršavanje jednokratnih komandi na Windows cilju. Na primer, uparivanje sa Metasploit-ovim modulom `web_delivery` omogućava izvršavanje PowerShell ciljanog obrnutog Meterpreter payload-a.
* Kreiranjem udaljene usluge na napadačevom računaru sa binPath postavljenim da pokrene pruženu komandu putem cmd.exe, moguće je uspešno izvršiti payload, postići povratni poziv i izvršavanje payload-a sa Metasploit slušaocem, čak i ako se pojave greške u odgovoru usluge.

### Primeri komandi

Kreiranje i pokretanje usluge moguće je postići sledećim komandama:

```bash
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```

Za dalje detalje pogledajte [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

## Reference

* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
