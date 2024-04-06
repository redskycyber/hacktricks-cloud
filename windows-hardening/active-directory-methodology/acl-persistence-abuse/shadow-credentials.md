# Shadow Credentials

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Uvod <a href="#id-3f17" id="id-3f17"></a>

**Proverite originalni post za** [**sve informacije o ovoj tehnici**](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)**.**

Ukratko: ako možete pisati u svojstvo **msDS-KeyCredentialLink** korisnika/računara, možete dobiti **NT hash tog objekta**.

U postu je opisan metod za podešavanje **javno-privatnih ključeva za autentifikaciju** kako bi se dobio jedinstveni **Service Ticket** koji uključuje NTLM hash cilja. Ovaj proces uključuje šifrovanje NTLM\_SUPPLEMENTAL\_CREDENTIAL unutar Privilege Attribute Certificate (PAC), koji se može dešifrovati.

### Zahtevi

Da biste primenili ovu tehniku, moraju se ispuniti određeni uslovi:

* Potreban je najmanje jedan Windows Server 2016 Domain Controller.
* Domain Controller mora imati instaliran digitalni sertifikat za server autentifikaciju.
* Active Directory mora biti na Windows Server 2016 Functional Level.
* Potreban je nalog sa delegiranim pravima za izmenu atributa msDS-KeyCredentialLink ciljnog objekta.

## Zloupotreba

Zloupotreba Key Trust-a za računarske objekte obuhvata korake koji idu dalje od dobijanja Ticket Granting Ticket (TGT) i NTLM hasha. Opcije uključuju:

1. Kreiranje **RC4 silver ticket-a** kako bi se delovalo kao privilegovani korisnici na ciljanom hostu.
2. Korišćenje TGT-a sa **S4U2Self** za impersonaciju **privilegovanih korisnika**, što zahteva izmene na Service Ticket-u kako bi se dodala klasa servisa imenu servisa.

Značajna prednost zloupotrebe Key Trust-a je ograničenje na privatni ključ koji generiše napadač, izbegavajući delegaciju potencijalno ranjivim nalozima i ne zahteva kreiranje računara, što može biti teško ukloniti.

## Alati

\### [**Whisker**](https://github.com/eladshamir/Whisker)

Zasnovan na DSInternals-u, pruža C# interfejs za ovaj napad. Whisker i njegov Python pandan, **pyWhisker**, omogućavaju manipulaciju atributom `msDS-KeyCredentialLink` kako bi se preuzela kontrola nad Active Directory nalozima. Ovi alati podržavaju različite operacije kao što su dodavanje, listanje, uklanjanje i brisanje ključnih akreditacija sa ciljnog objekta.

Funkcije **Whisker**-a uključuju:

* **Add**: Generiše par ključeva i dodaje ključne akreditacije.
* **List**: Prikazuje sve unose ključnih akreditacija.
* **Remove**: Briše određene ključne akreditacije.
* **Clear**: Briše sve ključne akreditacije, potencijalno ometajući legitimnu upotrebu WHfB.

```shell
Whisker.exe add /target:computername$ /domain:constoso.local /dc:dc1.contoso.local /path:C:\path\to\file.pfx /password:P@ssword1
```

### [pyWhisker](https://github.com/ShutdownRepo/pywhisker)

Proširuje funkcionalnost Whiskera na **UNIX-baziranim sistemima**, koristeći Impacket i PyDSInternals za sveobuhvatne mogućnosti iskorišćavanja, uključujući listanje, dodavanje i uklanjanje KeyCredentials, kao i njihovo uvoz i izvoz u JSON formatu.

```shell
python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "list"
```

### [ShadowSpray](https://github.com/Dec0ne/ShadowSpray/)

ShadowSpray ima za cilj da **iskoristi dozvole GenericWrite/GenericAll koje široke grupe korisnika mogu imati nad objektima domena** kako bi široko primenio ShadowCredentials. To podrazumeva prijavljivanje na domen, proveru funkcionalnog nivoa domena, enumeraciju objekata domena i pokušaj dodavanja KeyCredentials za dobijanje TGT-a i otkrivanje NT hash-a. Opcije za čišćenje i taktike rekurzivnog iskorišćavanja poboljšavaju njegovu korisnost.

## Reference

* [https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
* [https://github.com/eladshamir/Whisker](https://github.com/eladshamir/Whisker)
* [https://github.com/Dec0ne/ShadowSpray/](https://github.com/Dec0ne/ShadowSpray/)
* [https://github.com/ShutdownRepo/pywhisker](https://github.com/ShutdownRepo/pywhisker)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
