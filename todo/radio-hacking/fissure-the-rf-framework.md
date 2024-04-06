# FISSURE - The RF Framework

**Frekvencijski nezavisno razumevanje signala i obrnuto inženjerstvo zasnovano na SDR-u**

FISSURE je open-source RF i obrnuto inženjerski okvir dizajniran za sve nivoe veština sa kukama za otkrivanje i klasifikaciju signala, otkrivanje protokola, izvršavanje napada, manipulaciju IQ-om, analizu ranjivosti, automatizaciju i AI/ML. Okvir je izgrađen da promoviše brzu integraciju softverskih modula, radio uređaja, protokola, signala, skripti, tokova podataka, referentnog materijala i alata trećih strana. FISSURE je omogućivač radnog toka koji čuva softver na jednom mestu i omogućava timovima da se lako prilagode, deleći istu dokazanu konfiguraciju za određene Linux distribucije.

Okvir i alati koji su uključeni u FISSURE su dizajnirani da otkriju prisustvo RF energije, razumeju karakteristike signala, prikupljaju i analiziraju uzorke, razvijaju tehnike slanja i/ili ubrizgavanja, i kreiraju prilagođene terete ili poruke. FISSURE sadrži rastuću biblioteku informacija o protokolima i signalima koja pomaže u identifikaciji, kreiranju paketa i testiranju. Postoje mogućnosti za preuzimanje signalnih datoteka i izgradnju plejlista za simuliranje saobraćaja i testiranje sistema.

Prijateljski Python kod i korisnički interfejs omogućavaju početnicima da brzo nauče o popularnim alatima i tehnikama koje se odnose na RF i obrnuto inženjerstvo. Edukatori u oblasti sajber bezbednosti i inženjerstva mogu iskoristiti ugrađeni materijal ili koristiti okvir da demonstriraju svoje sopstvene primene u stvarnom svetu. Razvojni programeri i istraživači mogu koristiti FISSURE za svoje svakodnevne zadatke ili da predstave svoja najnovija rešenja širem auditorijumu. Kako svest i upotreba FISSURE-a rastu u zajednici, tako će se proširiti i obim njegovih mogućnosti i tehnologija koje obuhvata.

**Dodatne informacije**

* [AIS stranica](https://www.ainfosec.com/technologies/fissure/)
* [GRCon22 Slajdovi](https://events.gnuradio.org/event/18/contributions/246/attachments/84/164/FISSURE\_Poore\_GRCon22.pdf)
* [GRCon22 Rad](https://events.gnuradio.org/event/18/contributions/246/attachments/84/167/FISSURE\_Paper\_Poore\_GRCon22.pdf)
* [GRCon22 Video](https://www.youtube.com/watch?v=1f2umEKhJvE)
* [Transkript Hack Chat-a](https://hackaday.io/event/187076-rf-hacking-hack-chat/log/212136-hack-chat-transcript-part-1)

## Početak

**Podržano**

Postoje tri grane unutar FISSURE-a koje olakšavaju navigaciju kroz datoteke i smanjuju redundanciju koda. Grana Python2\_maint-3.7 sadrži kodnu bazu izgrađenu oko Python2, PyQt4 i GNU Radio 3.7; grana Python3\_maint-3.8 je izgrađena oko Python3, PyQt5 i GNU Radio 3.8; a grana Python3\_maint-3.10 je izgrađena oko Python3, PyQt5 i GNU Radio 3.10.

|   Operativni sistem  |    Grana FISSURE   |
| :------------------: | :----------------: |
|  Ubuntu 18.04 (x64)  | Python2\_maint-3.7 |
| Ubuntu 18.04.5 (x64) | Python2\_maint-3.7 |
| Ubuntu 18.04.6 (x64) | Python2\_maint-3.7 |
| Ubuntu 20.04.1 (x64) | Python3\_maint-3.8 |
| Ubuntu 20.04.4 (x64) | Python3\_maint-3.8 |
|  KDE neon 5.25 (x64) | Python3\_maint-3.8 |

**U toku (beta)**

Ovi operativni sistemi su još u beta statusu. Oni su u razvoju i poznato je da nekoliko funkcija nedostaje. Stavke u instalateru mogu biti u konfliktu sa postojećim programima ili neuspeli pri instalaciji dok se status ne ukloni.

|     Operativni sistem    |    Grana FISSURE    |
| :----------------------: | :-----------------: |
| DragonOS Focal (x86\_64) |  Python3\_maint-3.8 |
|    Ubuntu 22.04 (x64)    | Python3\_maint-3.10 |

Napomena: Određeni softverski alati ne rade na svakom operativnom sistemu. Pogledajte [Softver i konflikti](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Help/Markdown/SoftwareAndConflicts.md)

**Instalacija**

```
git clone https://github.com/ainfosec/FISSURE.git
cd FISSURE
git checkout <Python2_maint-3.7> or <Python3_maint-3.8> or <Python3_maint-3.10>
git submodule update --init
./install
```

Ovo će instalirati zavisnosti softvera PyQt potrebne za pokretanje GUI instalacija ako nisu pronađene.

Zatim, odaberite opciju koja najbolje odgovara vašem operativnom sistemu (trebalo bi da se automatski detektuje ako vaš OS odgovara jednoj od opcija).

|                                          Python2\_maint-3.7                                          |                                          Python3\_maint-3.8                                          |                                          Python3\_maint-3.10                                         |
| :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: |
| ![install1b](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1b.png) | ![install1a](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1a.png) | ![install1c](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1c.png) |

Preporučuje se instalacija FISSURE-a na čist operativni sistem kako bi se izbegli postojeći konflikti. Označite sve preporučene opcije (podrazumevana dugme) kako biste izbegli greške prilikom korišćenja različitih alata unutar FISSURE-a. Tokom instalacije će se pojaviti više prozora zahtevajući povišene privilegije i korisnička imena. Ako stavka sadrži odeljak "Provera" na kraju, instalater će pokrenuti naredbu koja sledi i označiti stavku za potvrdu zelenom ili crvenom bojom u zavisnosti od toga da li naredba proizvodi greške. Označene stavke bez odeljka "Provera" ostaće crne nakon instalacije.

![install2](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install2.png)

**Korišćenje**

Otvorite terminal i unesite:

```
fissure
```

## Detalji

**Komponente**

* Nadzorna tabla
* Centralni čvor (HIPRFISR)
* Identifikacija ciljnog signala (TSI)
* Otkrivanje protokola (PD)
* Graf toka i izvršitelj skripti (FGE)

![komponente](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/components.png)

**Mogućnosti**

| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/detector.png)_**Detektor signala**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/iq.png)_**Manipulacija IQ**_        | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/library.png)_**Pretraga signala**_         | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/pd.png)_**Prepoznavanje šablona**_ |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/attack.png)_**Napadi**_             | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/fuzzing.png)_**Faziranje**_         | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/archive.png)_**Plejliste signala**_        | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/gallery.png)_**Galerija slika**_   |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/packet.png)_**Oblikovanje paketa**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/scapy.png)_**Integracija Scapy-a**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/crc\_calculator.png)_**Kalkulator CRC-a**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/log.png)_**Evidentiranje**_        |

**Hardver**

Ovo je lista "podržanog" hardvera sa različitim nivoima integracije:

* USRP: X3xx, B2xx, B20xmini, USRP2, N2xx
* HackRF
* RTL2832U
* 802.11 adapteri
* LimeSDR
* bladeRF, bladeRF 2.0 micro
* Open Sniffer
* PlutoSDR

## Lekcije

FISSURE dolazi sa nekoliko korisnih vodiča kako biste se upoznali sa različitim tehnologijama i tehnikama. Mnogi od njih uključuju korake za korišćenje različitih alata koji su integrisani u FISSURE.

* [Lekcija 1: OpenBTS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson1\_OpenBTS.md)
* [Lekcija 2: Lua Dissectori](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson2\_LuaDissectors.md)
* [Lekcija 3: Sound eXchange](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson3\_Sound\_eXchange.md)
* [Lekcija 4: ESP ploče](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson4\_ESP\_Boards.md)
* [Lekcija 5: Praćenje radiosonda](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson5\_Radiosonde\_Tracking.md)
* [Lekcija 6: RFID](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson6\_RFID.md)
* [Lekcija 7: Tipovi podataka](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson7\_Data\_Types.md)
* [Lekcija 8: Prilagođeni GNU Radio blokovi](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson8\_Custom\_GNU\_Radio\_Blocks.md)
* [Lekcija 9: TPMS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson9\_TPMS.md)
* [Lekcija 10: Ham radio ispiti](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson10\_Ham\_Radio\_Exams.md)
* [Lekcija 11: Wi-Fi alati](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson11\_WiFi\_Tools.md)

## Plan

* [ ] Dodati više vrsta hardvera, RF protokola, parametara signala, alata za analizu
* [ ] Podržati više operativnih sistema
* [ ] Razviti materijal za časove o FISSURE-u (RF napadi, Wi-Fi, GNU Radio, PyQt, itd.)
* [ ] Kreirati uređaj za kondicioniranje signala, ekstraktor karakteristika i klasifikator signala sa mogućnostima izbora AI/ML tehnika
* [ ] Implementirati rekurzivne mehanizme demodulacije za dobijanje bitnog niza iz nepoznatih signala
* [ ] Prebaciti glavne komponente FISSURE-a na generičku šemu implementacije senzorskih čvorova

## Doprinose

Sugestije za poboljšanje FISSURE-a su veoma dobrodošle. Ostavite komentar na [Stranici za diskusije](https://github.com/ainfosec/FISSURE/discussions) ili na Discord serveru ako imate bilo kakve ideje u vezi sa sledećim:

* Predlozi novih funkcionalnosti i dizajnerskih promena
* Softverski alati sa koracima za instalaciju
* Nove lekcije ili dodatni materijal za postojeće lekcije
* RF protokoli od interesa
* Više hardvera i SDR tipova za integraciju
* Skripte za IQ analizu u Python-u
* Ispravke i poboljšanja instalacije

Doprinosi za poboljšanje FISSURE-a su od ključne važnosti za ubrzanje njegovog razvoja. Svaki vaš doprinos je veoma cenjen. Ako želite da doprinesete razvojem koda, molimo vas da klonirate repozitorijum i kreirate zahtev za povlačenje:

1. Klonirajte projekat
2. Kreirajte granu za funkcionalnost (`git checkout -b feature/AmazingFeature`)
3. Komitujte vaše promene (`git commit -m 'Dodajte neku neverovatnu funkcionalnost'`)
4. Pritisnite granu (`git push origin feature/AmazingFeature`)
5. Otvorite zahtev za povlačenje

Takođe je dobrodošlo kreiranje [Problema](https://github.com/ainfosec/FISSURE/issues) kako biste skrenuli pažnju na greške.

## Saradnja

Kontaktirajte Odeljenje za razvoj poslovanja Assured Information Security, Inc. (AIS) kako biste predložili i formalizovali mogućnosti saradnje u vezi sa FISSURE-om - bilo da se radi o posvećivanju vremena za integraciju vašeg softvera, razvoju rešenja za vaše tehničke izazove od strane talentovanih ljudi iz AIS-a ili integraciji FISSURE-a u druge platforme/aplikacije.

## Licenca

GPL-3.0

Za detalje o licenci, pogledajte datoteku LICENSE.

## Kontakt

Pridružite se Discord serveru: [https://discord.gg/JZDs5sgxcG](https://discord.gg/JZDs5sgxcG)

Pratite na Twitteru: [@FissureRF](https://twitter.com/fissurerf), [@AinfoSec](https://twitter.com/ainfosec)

Chris Poore - Assured Information Security, Inc. - poorec@ainfosec.com

Razvoj poslovanja - Assured Information Security, Inc. - bd@ainfosec.com

## Zasluge

Zahvaljujemo se i veoma smo zahvalni ovim programerima:

[Zasluge](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/CREDITS.md)

## Zahvalnost

Posebne zahvalnosti dr. Samuelu Mantravadiju i Josephu Reithu na njihovom doprinosu ovom projektu.
