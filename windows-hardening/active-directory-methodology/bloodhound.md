# BloodHound & Other AD Enum Tools

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## AD Explorer

[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) je deo Sysinternal Suite:

> Napredni pregledač i editor Active Directory (AD). Možete koristiti AD Explorer da biste lako navigirali kroz AD bazu podataka, definisali omiljene lokacije, pregledali osobine objekata i atribute bez otvaranja dijaloških okvira, menjali dozvole, pregledali šemu objekta i izvršavali složene pretrage koje možete sačuvati i ponovo izvršiti.

### Snimci

AD Explorer može kreirati snimke AD-a tako da ih možete proveriti offline.\
Može se koristiti za otkrivanje ranjivosti offline, ili za poređenje različitih stanja AD baze podataka tokom vremena.

Potrebno je korisničko ime, lozinka i pravac za povezivanje (potreban je bilo koji AD korisnik).

Da biste napravili snimak AD-a, idite na `File` --> `Create Snapshot` i unesite ime snimka.

## ADRecon

[**ADRecon**](https://github.com/adrecon/ADRecon) je alat koji izvlači i kombinuje razne artefakte iz AD okruženja. Informacije se mogu prikazati u **posebno formatiranom** Microsoft Excel **izveštaju** koji uključuje pregledne prikaze sa metrikama radi olakšane analize i pružanja celovite slike trenutnog stanja ciljnog AD okruženja.

```bash
# Run it
.\ADRecon.ps1
```

## BloodHound

Sa [https://github.com/BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound)

> BloodHound je jednostranična Javascript web aplikacija, izgrađena na vrhu [Linkurious](http://linkurio.us/), kompajlirana sa [Electron](http://electron.atom.io/), sa Neo4j bazom podataka koju napaja C# data kolektor.

BloodHound koristi teoriju grafova da otkrije skrivene i često nenamerne veze unutar Active Directory ili Azure okruženja. Napadači mogu koristiti BloodHound da lako identifikuju visoko kompleksne putanje napada koje bi inače bilo nemoguće brzo identifikovati. Odbrambeni timovi mogu koristiti BloodHound da identifikuju i eliminišu iste te putanje napada. I plavi i crveni timovi mogu koristiti BloodHound da lako steknu dublje razumevanje privilegovanih veza u Active Directory ili Azure okruženju.

Dakle, [Bloodhound](https://github.com/BloodHoundAD/BloodHound) je neverovatan alat koji može automatski nabrojati domen, sačuvati sve informacije, pronaći moguće putanje za eskalaciju privilegija i prikazati sve informacije koristeći grafove.

Bloodhound se sastoji od 2 glavna dela: **ingestori** i **aplikacija za vizualizaciju**.

**Ingestori** se koriste za **nabrojavanje domena i izvlačenje svih informacija** u formatu koji će aplikacija za vizualizaciju razumeti.

**Aplikacija za vizualizaciju koristi neo4j** da prikaže kako su sve informacije povezane i da prikaže različite načine za eskalaciju privilegija u domenu.

### Instalacija

Nakon stvaranja BloodHound CE, ceo projekat je ažuriran radi lakšeg korišćenja sa Dockerom. Najlakši način za početak je korišćenje prekonfigurisane Docker Compose konfiguracije.

1. Instalirajte Docker Compose. Ovo bi trebalo da bude uključeno u instalaciju [Docker Desktop](https://www.docker.com/products/docker-desktop/).
2. Pokrenite:

```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```

3. Pronađite nasumično generisanu lozinku u terminalnom izlazu Docker Compose-a.
4. U pretraživaču, idite na http://localhost:8080/ui/login. Prijavite se sa korisničkim imenom admin i nasumično generisanom lozinkom iz logova.

Nakon toga ćete morati da promenite nasumično generisanu lozinku i imaćete novi interfejs spreman, sa kojeg možete direktno preuzeti ingestore.

### SharpHound

Imaju nekoliko opcija, ali ako želite da pokrenete SharpHound sa računara koji je pridružen domenu, koristeći trenutnog korisnika i izvučete sve informacije, možete uraditi sledeće:

```
./SharpHound.exe --CollectionMethods All
Invoke-BloodHound -CollectionMethod All
```

> Više informacija o **CollectionMethod** i petlji sesije možete pročitati [ovde](https://support.bloodhoundenterprise.io/hc/en-us/articles/17481375424795-All-SharpHound-Community-Edition-Flags-Explained)

Ako želite da izvršite SharpHound koristeći različite akreditive, možete kreirati CMD netonly sesiju i pokrenuti SharpHound iz nje:

```
runas /netonly /user:domain\user "powershell.exe -exec bypass"
```

[**Saznajte više o Bloodhound-u na ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)

## Group3r

[**Group3r**](https://github.com/Group3r/Group3r) je alat za pronalaženje **ranjivosti** u Active Directory-u povezanih sa **Group Policy**-jem.\
Morate **pokrenuti group3r** sa računara unutar domena koristeći **bilo koji korisnički nalog domena**.

```bash
group3r.exe -f <filepath-name.log>
# -s sends results to stdin
# -f send results to file
```

## PingCastle

[**PingCastle**](https://www.pingcastle.com/documentation/) **procenjuje sigurnosni položaj AD okruženja** i pruža lep **izveštaj** sa grafikonima.

Da biste ga pokrenuli, možete izvršiti binarnu datoteku `PingCastle.exe` i ona će pokrenuti **interaktivnu sesiju** koja prikazuje meni sa opcijama. Podrazumevana opcija za korišćenje je **`healthcheck`** koja će uspostaviti osnovni **pregled** domena i pronaći **pogrešne konfiguracije** i **ranjivosti**.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
