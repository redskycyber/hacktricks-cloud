# AppendData/AddSubdirectory permission over service registry

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite videti **vašu kompaniju oglašenu na HackTricks-u** ili **preuzeti HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Originalni post je** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

### Rezime

Dve registarske ključeve je moguće pisati od strane trenutnog korisnika:

* **`HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`**
* **`HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`**

Predloženo je proveriti dozvole servisa **RpcEptMapper** koristeći **regedit GUI**, posebno prozor **Advanced Security Settings** i karticu **Effective Permissions**. Ovaj pristup omogućava procenu dodeljenih dozvola specifičnim korisnicima ili grupama bez potrebe za pregledanjem svakog pojedinačnog Access Control Entry (ACE).

Prikazane su dozvole dodeljene korisniku sa niskim privilegijama, među kojima je značajna dozvola **Create Subkey**. Ova dozvola, takođe poznata kao **AppendData/AddSubdirectory**, odgovara nalazima skripte.

Primećeno je da nije moguće direktno menjati određene vrednosti, ali je moguće kreirati nove podključeve. Primer je dat pokušaja izmene vrednosti **ImagePath**, koji je rezultirao porukom o odbijanju pristupa.

Uprkos ovim ograničenjima, identifikovana je mogućnost eskalacije privilegija putem mogućnosti iskorišćavanja podključa **Performance** unutar registarske strukture servisa **RpcEptMapper**, podključa koji nije prisutan podrazumevano. Ovo bi omogućilo registraciju DLL fajlova i praćenje performansi.

Konsultovana je dokumentacija o podključu **Performance** i njegovoj upotrebi za praćenje performansi, što je dovelo do razvoja DLL-a kao dokaza koncepta. Ovaj DLL, koji demonstrira implementaciju funkcija **OpenPerfData**, **CollectPerfData** i **ClosePerfData**, testiran je putem **rundll32**, potvrđujući njegovu operativnu uspešnost.

Cilj je bio prisiliti **RPC Endpoint Mapper servis** da učita izrađeni Performance DLL. Posmatranjem je otkriveno da izvršavanje WMI klasnih upita koji se odnose na Performance Data putem PowerShell-a rezultira kreiranjem log fajla, omogućavajući izvršavanje proizvoljnog koda pod **LOCAL SYSTEM** kontekstom, čime se dodeljuju povišene privilegije.

Istaknuta je postojanost i potencijalne posledice ove ranjivosti, naglašavajući njenu relevantnost za post-eksploatacijske strategije, lateralno kretanje i izbegavanje antivirusnih/EDR sistema.

Iako je ranjivost prvobitno otkrivena nenamerno putem skripte, naglašeno je da je njeno iskorišćavanje ograničeno na zastarele verzije Windows-a (npr. **Windows 7 / Server 2008 R2**) i zahteva lokalni pristup.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite videti **vašu kompaniju oglašenu na HackTricks-u** ili **preuzeti HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
