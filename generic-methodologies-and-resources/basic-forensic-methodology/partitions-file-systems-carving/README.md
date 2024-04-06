# Partitions/File Systems/Carving

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Particije

Tvrdi disk ili **SSD disk mogu sadržati različite particije** sa ciljem fizičkog razdvajanja podataka.\
**Minimalna** jedinica diska je **sektor** (obično sastavljen od 512B). Dakle, veličina svake particije mora biti višekratnik te veličine.

### MBR (master Boot Record)

Nalazi se u **prvom sektoru diska nakon 446B boot koda**. Taj sektor je bitan jer pokazuje računaru šta i odakle treba da se montira particija.\
Dozvoljava do **4 particije** (najviše **samo 1** može biti aktivna/**bootable**). Međutim, ako vam je potrebno više particija, možete koristiti **proširene particije**. **Poslednji bajt** ovog prvog sektora je potpis boot zapisa **0x55AA**. Samo jedna particija može biti označena kao aktivna.\
MBR dozvoljava **maksimalno 2.2TB**.

![](<../../../.gitbook/assets/image (489).png>)

![](<../../../.gitbook/assets/image (490).png>)

Od **bajtova 440 do 443** MBR-a možete pronaći **Windows Disk Signature** (ako se koristi Windows). Logičko slovo drajva tvrdog diska zavisi od Windows Disk Signature-a. Menjanje ovog potpisa može sprečiti Windows da se podigne (alat: [**Active Disk Editor**](https://www.disk-editor.org/index.html)**)**.

![](<../../../.gitbook/assets/image (493).png>)

**Format**

| Offset      | Dužina     | Stavka            |
| ----------- | ---------- | ----------------- |
| 0 (0x00)    | 446(0x1BE) | Boot kod          |
| 446 (0x1BE) | 16 (0x10)  | Prva particija    |
| 462 (0x1CE) | 16 (0x10)  | Druga particija   |
| 478 (0x1DE) | 16 (0x10)  | Treća particija   |
| 494 (0x1EE) | 16 (0x10)  | Četvrta particija |
| 510 (0x1FE) | 2 (0x2)    | Potpis 0x55 0xAA  |

**Format Zapisa Particije**

| Offset    | Dužina   | Stavka                                                     |
| --------- | -------- | ---------------------------------------------------------- |
| 0 (0x00)  | 1 (0x01) | Aktivna oznaka (0x80 = bootable)                           |
| 1 (0x01)  | 1 (0x01) | Početna glava                                              |
| 2 (0x02)  | 1 (0x01) | Početni sektor (bitovi 0-5); gornji bitovi cilindra (6- 7) |
| 3 (0x03)  | 1 (0x01) | Najnižih 8 bitova cilindra početka                         |
| 4 (0x04)  | 1 (0x01) | Kod tipa particije (0x83 = Linux)                          |
| 5 (0x05)  | 1 (0x01) | Krajnja glava                                              |
| 6 (0x06)  | 1 (0x01) | Krajnji sektor (bitovi 0-5); gornji bitovi cilindra (6- 7) |
| 7 (0x07)  | 1 (0x01) | Najnižih 8 bitova cilindra kraja                           |
| 8 (0x08)  | 4 (0x04) | Sektori pre particije (little endian)                      |
| 12 (0x0C) | 4 (0x04) | Sektori u particiji                                        |

Da biste montirali MBR u Linux-u prvo morate dobiti početni offset (možete koristiti `fdisk` i komandu `p`)

![](https://github.com/carlospolop/hacktricks/blob/rs/.gitbook/assets/image%20\(413\)%20\(3\)%20\(3\)%20\(3\)%20\(2\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(12\).png)

Zatim koristite sledeći kod

```bash
#Mount MBR in Linux
mount -o ro,loop,offset=<Bytes>
#63x512 = 32256Bytes
mount -o ro,loop,offset=32256,noatime /path/to/image.dd /media/part/
```

**LBA (Logical block addressing)**

**Logičko blok adresiranje** (**LBA**) je uobičajena šema korišćena za **specifikaciju lokacije blokova** podataka smeštenih na računarskim uređajima za skladištenje, uglavnom sekundarnim skladišnim sistemima poput hard disk drajvova. LBA je posebno jednostavna linearna adresna šema; **blokovi se lociraju pomoću celobrojnog indeksa**, pri čemu je prvi blok LBA 0, drugi LBA 1, i tako dalje.

### GPT (GUID Partition Table)

GUID Partition Table, poznata kao GPT, preferira se zbog svojih unapređenih mogućnosti u poređenju sa MBR (Master Boot Record). Karakteristična po svom **globalno jedinstvenom identifikatoru** za particije, GPT se ističe na nekoliko načina:

* **Lokacija i Veličina**: I GPT i MBR počinju na **sektoru 0**. Međutim, GPT radi na **64 bita**, za razliku od MBR-ovih 32 bita.
* **Ograničenja particija**: GPT podržava do **128 particija** na Windows sistemima i može da primi do **9.4ZB** podataka.
* **Imena particija**: Omogućava mogućnost imenovanja particija sa do 36 Unicode karaktera.

**Otpornost i Obnova Podataka**:

* **Redundantnost**: Za razliku od MBR-a, GPT ne ograničava particionisanje i podatke o podizanju na jednom mestu. Ona replicira ove podatke širom diska, poboljšavajući integritet i otpornost podataka.
* **Ciklična Redundantna Provera (CRC)**: GPT koristi CRC kako bi osigurala integritet podataka. Aktivno nadgleda korupciju podataka, i kada je detektovana, GPT pokušava da povrati oštećene podatke sa druge lokacije na disku.

**Zaštitni MBR (LBA0)**:

* GPT održava kompatibilnost unazad putem zaštitnog MBR-a. Ova funkcija se nalazi u prostoru za stari MBR, ali je dizajnirana da spreči starije MBR-bazirane alatke da greškom prepišu GPT diskove, čime se čuva integritet podataka na GPT-formatiranim diskovima.

![https://upload.wikimedia.org/wikipedia/commons/thumb/0/07/GUID\_Partition\_Table\_Scheme.svg/800px-GUID\_Partition\_Table\_Scheme.svg.png](<../../../.gitbook/assets/image (491).png>)

**Hibridni MBR (LBA 0 + GPT)**

[Od Vikipedije](https://en.wikipedia.org/wiki/GUID\_Partition\_Table)

U operativnim sistemima koji podržavaju **GPT bazirano podizanje putem BIOS** servisa umesto EFI, prvi sektor se može koristiti za skladištenje prvog koraka **koda podizanja** (bootloader), ali **modifikovanog** da prepozna **GPT** **particije**. Bootloader u MBR-u ne sme pretpostaviti veličinu sektora od 512 bajtova.

**Zaglavlje particionog tabele (LBA 1)**

[Od Vikipedije](https://en.wikipedia.org/wiki/GUID\_Partition\_Table)

Zaglavlje particione tabele definiše upotrebljive blokove na disku. Takođe definiše broj i veličinu unosa particija koji čine particionu tabelu (ofseti 80 i 84 u tabeli).

| Ofset     | Dužina     | Sadržaj                                                                                                                                                                          |
| --------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0 (0x00)  | 8 bajtova  | Potpis ("EFI PART", 45h 46h 49h 20h 50h 41h 52h 54h ili 0x5452415020494645ULL[ ](https://en.wikipedia.org/wiki/GUID\_Partition\_Table#cite\_note-8)na malo-endijanskim mašinama) |
| 8 (0x08)  | 4 bajta    | Revizija 1.0 (00h 00h 01h 00h) za UEFI 2.8                                                                                                                                       |
| 12 (0x0C) | 4 bajta    | Veličina zaglavlja u malom endijanu (u bajtovima, obično 5Ch 00h 00h 00h ili 92 bajta)                                                                                           |
| 16 (0x10) | 4 bajta    | [CRC32](https://en.wikipedia.org/wiki/CRC32) zaglavlja (ofset +0 do veličine zaglavlja) u malom endijanu, pri čemu je ovo polje nula tokom računanja                             |
| 20 (0x14) | 4 bajta    | Rezervisano; mora biti nula                                                                                                                                                      |
| 24 (0x18) | 8 bajtova  | Trenutni LBA (lokacija ovog kopiranog zaglavlja)                                                                                                                                 |
| 32 (0x20) | 8 bajtova  | Rezervisano LBA (lokacija drugog kopiranog zaglavlja)                                                                                                                            |
| 40 (0x28) | 8 bajtova  | Prvi upotrebljivi LBA za particije (poslednji LBA primarne particione tabele + 1)                                                                                                |
| 48 (0x30) | 8 bajtova  | Poslednji upotrebljivi LBA (prvi LBA sekundarne particione tabele − 1)                                                                                                           |
| 56 (0x38) | 16 bajtova | Disk GUID u mešovitom endijanu                                                                                                                                                   |
| 72 (0x48) | 8 bajtova  | Početni LBA niza unosa particija (uvek 2 u primarnom kopiranju)                                                                                                                  |
| 80 (0x50) | 4 bajta    | Broj unosa particija u nizu                                                                                                                                                      |
| 84 (0x54) | 4 bajta    | Veličina jednog unosa particije (obično 80h ili 128)                                                                                                                             |
| 88 (0x58) | 4 bajta    | CRC32 niza unosa particija u malom endijanu                                                                                                                                      |
| 92 (0x5C) | \*         | Rezervisano; moraju biti nule za ostatak bloka (420 bajtova za veličinu sektora od 512 bajtova; ali može biti više sa većim veličinama sektora)                                  |

**Unosi particija (LBA 2–33)**

| Format unosa particije GUID |            |                                                                                                                       |
| --------------------------- | ---------- | --------------------------------------------------------------------------------------------------------------------- |
| Ofset                       | Dužina     | Sadržaj                                                                                                               |
| 0 (0x00)                    | 16 bajtova | [GUID particije tipa](https://en.wikipedia.org/wiki/GUID\_Partition\_Table#Partition\_type\_GUIDs) (mešoviti endijan) |
| 16 (0x10)                   | 16 bajtova | Jedinstveni GUID particije (mešoviti endijan)                                                                         |
| 32 (0x20)                   | 8 bajtova  | Prvi LBA ([mali endijan](https://en.wikipedia.org/wiki/Little\_endian))                                               |
| 40 (0x28)                   | 8 bajtova  | Poslednji LBA (inkluzivno, obično neparan)                                                                            |
| 48 (0x30)                   | 8 bajtova  | Zastavice atributa (npr. bit 60 označava samo za čitanje)                                                             |
| 56 (0x38)                   | 72 bajta   | Ime particije (36 [UTF-16](https://en.wikipedia.org/wiki/UTF-16)LE jedinica koda)                                     |

**Tipovi Particija**

![](<../../../.gitbook/assets/image (492).png>)

Više tipova particija na [https://en.wikipedia.org/wiki/GUID\_Partition\_Table](https://en.wikipedia.org/wiki/GUID\_Partition\_Table)

### Inspekcija

Nakon montiranja forenzičke slike sa [**ArsenalImageMounter**](https://arsenalrecon.com/downloads/), možete pregledati prvi sektor koristeći Windows alatku [**Active Disk Editor**](https://www.disk-editor.org/index.html)**.** Na sledećoj slici detektovan je **MBR** na **sektoru 0** i interpretiran:

![](<../../../.gitbook/assets/image (494).png>)

Da je to bio **GPT sto umesto MBR-a**, trebalo bi da se pojavi potpis _EFI PART_ u **sektoru 1** (koji je prazan na prethodnoj slici).

## Fajl-sistemi

### Lista Windows fajl-sistema

* **FAT12/16**: MSDOS, WIN95/98/NT/200
* **FAT32**: 95/2000/XP/2003/VISTA/7/8/10
* **ExFAT**: 2008/2012/2016/VISTA/7/8/10
* **NTFS**: XP/2003/2008/2012/VISTA/7/8/10
* **ReFS**: 2012/2016

### FAT

**FAT (File Allocation Table)** fajl-sistem je dizajniran oko svoje osnovne komponente, tabele alokacije fajlova, koja se nalazi na početku zapremine. Ovaj sistem štiti podatke održavanjem **dve kopije** tabele, obezbeđujući integritet podataka čak i ako je jedna oštećena. Tabela, zajedno sa korenskim folderom, mora biti na **fiksnom mestu**, ključnom za proces pokretanja sistema.

Osnovna jedinica skladištenja fajl-sistema je **klaster, obično 512B**, koji se sastoji od više sektora. FAT se razvijao kroz verzije:

* **FAT12**, podržava 12-bitne adrese klastera i upravlja do 4078 klastera (4084 sa UNIX-om).
* **FAT16**, unapređenje na 16-bitne adrese, čime se može smestiti do 65.517 klastera.
* **FAT32**, dalje napredovanje sa 32-bitnim adresama, omogućavajući impresivnih 268.435.456 klastera po zapremini.

Značajno ograničenje kroz verzije FAT-a je **maksimalna veličina fajla od 4GB**, nametnuta 32-bitnim poljem korišćenim za skladištenje veličine fajla.

Ključne komponente korenskog direktorijuma, posebno za FAT12 i FAT16, uključuju:

* **Ime fajla/foldera** (do 8 karaktera)
* **Atributi**
* **Datumi kreiranja, modifikacije i poslednjeg pristupa**
* **Adresa FAT tabele** (ukazujući na početni klaster fajla)
* **Veličina fajla**

### EXT

**Ext2** je najčešći fajl-sistem za **particije bez dnevnika** (**particije koje se retko menjaju**), poput boot particije. **Ext3/4** su **sa dnevnikom** i obično se koriste za **ostale particije**.

## **Metapodaci**

Neki fajlovi sadrže metapodatke. Ove informacije se odnose na sadržaj fajla koji ponekad može biti zanimljiv analitičaru jer, u zavisnosti od tipa fajla, može sadržati informacije poput:

* Naslov
* Korišćena verzija MS Office-a
* Autor
* Datumi kreiranja i poslednje modifikacije
* Model kamere
* GPS koordinate
* Informacije o slici

Možete koristiti alate poput [**exiftool**](https://exiftool.org) i [**Metadiver**](https://www.easymetadata.com/metadiver-2/) da biste dobili metapodatke fajla.

## **Obnova obrisanih fajlova**

### Evidentirani obrisani fajlovi

Kao što je već viđeno, postoji nekoliko mesta gde se fajl i dalje čuva nakon što je "obrisan". To je zato što brisanje fajla iz fajl-sistema obično označava kao obrisano, ali podaci nisu dirani. Zatim je moguće pregledati registre fajlova (poput MFT-a) i pronaći obrisane fajlove.

Takođe, OS obično čuva mnogo informacija o promenama fajl-sistema i rezervnim kopijama, pa je moguće pokušati ih koristiti za obnovu fajla ili što više informacija.

{% content-ref url="file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### **Izvlačenje fajlova**

**Izvlačenje fajlova** je tehnika koja pokušava **pronaći fajlove u masi podataka**. Postoje 3 glavna načina rada alata poput ovog: **Na osnovu zaglavlja i podnožja tipova fajlova**, na osnovu **strukture tipova fajlova** i na osnovu **sadržaja** samog fajla.

Imajte na umu da ova tehnika **ne funkcioniše za povrat fragmentiranih fajlova**. Ako fajl **nije smešten u susednim sektorima**, tada ova tehnika neće moći da ga pronađe ili bar deo njega.

Postoje različiti alati koje možete koristiti za izvlačenje fajlova, navodeći tipove fajlova koje želite pretraživati.

{% content-ref url="file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### Izvlačenje podataka iz **struja**

Izvlačenje podataka iz struja slično je izvlačenju fajlova, ali **umesto traženja kompletnih fajlova, traži zanimljive fragmente** informacija.\
Na primer, umesto traženja kompletnog fajla koji sadrži evidentirane URL-ove, ovom tehnikom će se tražiti URL-ovi.

{% content-ref url="file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### Bezbedno brisanje

Očigledno, postoje načini za **"sigurno" brisanje fajlova i delova zapisa o njima**. Na primer, moguće je **prepisati sadržaj** fajla sa beskorisnim podacima nekoliko puta, a zatim **ukloniti** zapise iz **$MFT** i **$LOGFILE** o fajlu, i **ukloniti kopije senki zapisa**.\
Možda ćete primetiti da čak i nakon sprovođenja te radnje, postoji **još delova gde je postojanje fajla evidentirano**, što je tačno, a deo posla forenzičara je da ih pronađe.

## Reference

* [https://en.wikipedia.org/wiki/GUID\_Partition\_Table](https://en.wikipedia.org/wiki/GUID\_Partition\_Table)
* [http://ntfs.com/ntfs-permissions.htm](http://ntfs.com/ntfs-permissions.htm)
* [https://www.osforensics.com/faqs-and-tutorials/how-to-scan-ntfs-i30-entries-deleted-files.html](https://www.osforensics.com/faqs-and-tutorials/how-to-scan-ntfs-i30-entries-deleted-files.html)
* [https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service)
* **iHackLabs Certified Digital Forensics Windows**
