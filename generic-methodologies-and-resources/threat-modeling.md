# Threat Modeling

## Model pretnje

Dobrodošli u sveobuhvatni vodič HackTricks-a o modeliranju pretnji! Krenite u istraživanje ovog ključnog aspekta kibernetičke sigurnosti, gde identifikujemo, razumemo i strategiziramo protiv potencijalnih ranjivosti u sistemu. Ova nit služi kao vodič korak po korak, pun stvarnih primera, korisnog softvera i lako razumljivih objašnjenja. Idealno za početnike i iskusne stručnjake koji žele ojačati svoju kibernetičku odbranu.

### Često korišćeni scenariji

1. **Razvoj softvera**: Kao deo Secure Software Development Life Cycle (SSDLC), modeliranje pretnji pomaže u **identifikaciji potencijalnih izvora ranjivosti** u ranim fazama razvoja.
2. **Pentestiranje**: Okvir Penetration Testing Execution Standard (PTES) zahteva **modeliranje pretnji kako bi se razumela ranjivost sistema** pre sprovođenja testa.

### Model pretnji ukratko

Model pretnji se obično predstavlja kao dijagram, slika ili neka druga vrsta vizualne ilustracije koja prikazuje planiranu arhitekturu ili postojeću izgradnju aplikacije. Sličan je **dijagramu protoka podataka**, ali ključna razlika leži u njegovom dizajnu usmerenom na sigurnost.

Modeli pretnji često sadrže elemente označene crvenom bojom, simbolizujući potencijalne ranjivosti, rizike ili prepreke. Da bi se olakšao proces identifikacije rizika, koristi se CIA (Poverljivost, Integritet, Dostupnost) trijad, koja je osnova mnogih metodologija modeliranja pretnji, pri čemu je STRIDE jedna od najčešćih. Međutim, izbor metodologije može varirati u zavisnosti od specifičnog konteksta i zahteva.

### CIA trijad

CIA trijad je široko priznat model u oblasti informacione sigurnosti, koji označava Poverljivost, Integritet i Dostupnost. Ova tri stuba čine osnovu mnogih sigurnosnih mera i politika, uključujući metodologije modeliranja pretnji.

1. **Poverljivost**: Osiguravanje da podaci ili sistem ne budu dostupni neovlašćenim osobama. Ovo je centralni aspekt sigurnosti, koji zahteva odgovarajuće kontrole pristupa, enkripciju i druge mere kako bi se sprečilo curenje podataka.
2. **Integritet**: Tačnost, doslednost i pouzdanost podataka tokom njihovog životnog ciklusa. Ovaj princip osigurava da podaci ne budu izmenjeni ili manipulisani od strane neovlašćenih strana. Često uključuje kontrolne zbirke, heširanje i druge metode provere podataka.
3. **Dostupnost**: Ovo osigurava da podaci i usluge budu dostupni ovlašćenim korisnicima kada su im potrebni. Često uključuje redundancu, toleranciju na greške i konfiguracije visoke dostupnosti kako bi se sistemi održavali čak i u slučaju prekida.

### Metodologije modeliranja pretnji

1. **STRIDE**: Razvijen od strane Microsoft-a, STRIDE je akronim za **Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service i Elevation of Privilege**. Svaka kategorija predstavlja vrstu pretnje, a ova metodologija se često koristi u fazi dizajna programa ili sistema radi identifikacije potencijalnih pretnji.
2. **DREAD**: Ovo je još jedna metodologija od Microsoft-a koja se koristi za procenu rizika identifikovanih pretnji. DREAD označava **Potencijal štete, Reproduktivnost, Iskorišćivost, Pogođeni korisnici i Otkrivanje**. Svaki od ovih faktora se ocenjuje, a rezultat se koristi za prioritetizaciju identifikovanih pretnji.
3. **PASTA** (Process for Attack Simulation and Threat Analysis): Ovo je sedmostepena, **rizik-orientisana** metodologija. Uključuje definisanje i identifikaciju sigurnosnih ciljeva, kreiranje tehničkog opsega, dekompoziciju aplikacije, analizu pretnji, analizu ranjivosti i procenu rizika/triage.
4. **Trike**: Ovo je metodologija zasnovana na riziku koja se fokusira na odbranu imovine. Polazi od **perspektive upravljanja rizikom** i posmatra pretnje i ranjivosti u tom kontekstu.
5. **VAST** (Visual, Agile i Simple Threat modeling): Ovaj pristup ima za cilj da bude pristupačniji i integriše se u okruženja agilnog razvoja. Kombinuje elemente drugih metodologija i fokusira se na **vizualno prikazivanje pretnji**.
6. **OCTAVE** (Operationally Critical Threat, Asset i Vulnerability Evaluation): Razvijen od strane CERT Coordination Center-a, ovaj okvir je usmeren na **procenu rizika organizacije, a ne specifičnih sistema ili softvera**.

## Alati

Postoji nekoliko alata i softverskih rešenja dostupnih koji mogu **pomoći** u kreiranju i upravljanju modelima pretnji. Evo nekoliko koje biste mogli razmotriti.

### [SpiderSuite](https://github.com/3nock/SpiderSuite)

Napredni prenosivi i višefunkcionalni GUI web spider/crawler za profesionalce u oblasti kibernetičke sigurnosti. Spider Suite se može koristiti za mapiranje i analizu površine napada.

**Upotreba**

1. Izaberite URL i izvršite pretragu

<figure><img src="../.gitbook/assets/threatmodel_spidersuite_1.png" alt=""><figcaption></figcaption></figure>

2. Prikaz grafa

<figure><img src="../.gitbook/assets/threatmodel_spidersuite_2.png" alt=""><figcaption></figcaption></figure>

### [OWASP Threat Dragon](https://github.com/OWASP/threat-dragon/releases)

Projekat otvorenog koda OWASP-a, Threat Dragon je web i desktop aplikacija koja uključuje dijagram sistema, kao i pravila za automatsko generisanje pretnji/mitigacija.

**Upotreba**

1. Kreirajte novi projekat

<figure><img src="../.gitbook/assets/create_new_project_1.jpg" alt=""><figcaption></figcaption></figure>

Ponekad može izgledati ovako:

<figure><img src="../.gitbook/assets/1_threatmodel_create_project.jpg" alt=""><figcaption></figcaption></figure>

2. Pokrenite novi projekat

<figure><img src="../.gitbook/assets/launch_new_project_2.jpg" alt=""><figcaption></figcaption></figure>

3. Sačuvajte novi projekat

<figure><img src="../.gitbook/assets/save_new_project.jpg" alt=""><figcaption></figcaption></figure>

4. Kreirajte svoj model

Možete koristiti alate poput SpiderSuite Crawler-a da vam pruže inspiraciju, osnovni model bi izgledao nešto poput ovoga

<figure><img src="../.gitbook/assets/0_basic_threat_model.jpg" alt=""><figcaption></figcaption></figure>

Samo malo objašnjenja o entitetima:

* Proces (Sam entitet kao što je Web server ili web funkcionalnost)
* Akter (Osoba kao što je posetilac veb sajta, korisnik ili administrator)
* Linija protoka podataka (Indikator interakcije)
* Granica poverenja (Različiti mrežni segmenti ili opsezi.)
* Skladište (Mesta gde se podaci čuvaju, kao što su baze podataka)

5. Kreirajte pretnju (Korak 1)

Prvo morate odabrati sloj na koji želite dodati pretnju

<figure><img src="../.gitbook/assets/3_threatmodel_chose-threat-layer.jpg" alt=""><figcaption></figcaption></figure>

Sada možete kreirati pretnju

<figure><img src="../.gitbook/assets/4_threatmodel_create-threat.jpg" alt=""><figcaption></figcaption></figure>

Imajte na umu da postoji razlika između pretnji aktera i pretnji procesa. Ako biste dodali pretnju akteru, moći ćete izabrati samo "Spoofing" i "Repudiation". Međutim, u našem primeru dodajemo pretnju procesnom entitetu, pa ćemo videti ovo u okviru za kreiranje pretnje:
