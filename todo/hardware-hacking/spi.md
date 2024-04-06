# SPI

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovne informacije

SPI (Serial Peripheral Interface) je sinhroni serijski komunikacioni protokol koji se koristi u ugrađenim sistemima za komunikaciju na kratkim rastojanjima između IC-ova (Integrisanih kola). SPI komunikacioni protokol koristi arhitekturu master-slave koju orkestrira Clock i Chip Select Signal. Arhitektura master-slave sastoji se od mastera (obično mikroprocesora) koji upravlja spoljnim perifernim uređajima poput EEPROM-a, senzora, kontrolnih uređaja, itd. koji se smatraju robovima.

Više robova može biti povezano sa masterom, ali robovi ne mogu komunicirati međusobno. Robovi se upravljaju sa dva pina, clock i chip select. Budući da je SPI sinhroni komunikacioni protokol, ulazni i izlazni pinovi prate clock signale. Chip select se koristi od strane mastera da odabere roba i interaguje sa njim. Kada je chip select visok, uređaj roba nije izabran, dok je kada je nizak, čip je izabran i master će interagovati sa robovima.

MOSI (Master Out, Slave In) i MISO (Master In, Slave Out) su odgovorni za slanje i primanje podataka. Podaci se šalju uređaju roba putem pina MOSI dok je chip select nizak. Ulazni podaci sadrže instrukcije, memorijske adrese ili podatke prema listi podataka dobavljača uređaja roba. Nakon važećeg unosa, pin MISO je odgovoran za slanje podataka masteru. Izlazni podaci se šalju tačno na sledećem ciklusu sata nakon završetka unosa. Pin MISO prenosi podatke dok se podaci potpuno ne prenesu ili master postavi chip select pin na visok (u tom slučaju, roba će prestati sa slanjem i master neće slušati nakon tog ciklusa sata).

## Dumpovanje fleša

### Bus Pirate + flashrom

![](<../../.gitbook/assets/image (201).png>)

Imajte na umu da čak i ako PINOUT Pirate Bus-a pokazuje pinove za **MOSI** i **MISO** za povezivanje na SPI, neki SPI-ovi mogu pokazati pinove kao DI i DO. **MOSI -> DI, MISO -> DO**

![](<../../.gitbook/assets/image (648) (1) (1).png>)

U Windows-u ili Linux-u možete koristiti program [**`flashrom`**](https://www.flashrom.org/Flashrom) za dumpovanje sadržaja fleš memorije pokretanjem nečega poput:

```bash
# In this command we are indicating:
# -VV Verbose
# -c <chip> The chip (if you know it better, if not, don'tindicate it and the program might be able to find it)
# -p <programmer> In this case how to contact th chip via the Bus Pirate
# -r <file> Image to save in the filesystem
flashrom -VV -c "W25Q64.V" -p buspirate_spi:dev=COM3 -r flash_content.img
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
