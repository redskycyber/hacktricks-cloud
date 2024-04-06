# UART

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovne informacije

UART je serijski protokol, što znači da prenosi podatke između komponenti po jedan bit u isto vreme. Za razliku od toga, paralelni komunikacioni protokoli prenose podatke istovremeno kroz više kanala. Uobičajeni serijski protokoli uključuju RS-232, I2C, SPI, CAN, Ethernet, HDMI, PCI Express i USB.

Općenito, linija se drži visoko (na logičkoj vrednosti 1) dok je UART u stanju mirovanja. Zatim, da bi signalizirao početak prenosa podataka, predajnik šalje startni bit prijemniku, tokom kojeg se signal drži nisko (na logičkoj vrednosti 0). Zatim, predajnik šalje pet do osam podatkovnih bitova koji sadrže stvarnu poruku, praćenu opcionalnim bitom parnosti i jednim ili dva stop bita (sa logičkom vrednošću 1), zavisno od konfiguracije. Bit parnosti, koji se koristi za proveru grešaka, retko se viđa u praksi. Stop bit (ili bitovi) označavaju kraj prenosa.

Najčešću konfiguraciju zovemo 8N1: osam podatkovnih bitova, bez parnosti i jedan stop bit. Na primer, ako bismo želeli poslati karakter C, ili 0x43 u ASCII, u UART konfiguraciji 8N1, poslali bismo sledeće bitove: 0 (startni bit); 0, 1, 0, 0, 0, 0, 1, 1 (vrednost 0x43 u binarnom obliku) i 0 (stop bit).

![](<../../.gitbook/assets/image (648) (1) (1) (1) (1).png>)

Hardverski alati za komunikaciju sa UART-om:

* USB-za-serijski adapter
* Adapteri sa čipovima CP2102 ili PL2303
* Višenamenski alat poput: Bus Pirate, Adafruit FT232H, Shikra ili Attify Badge

### Identifikacija UART portova

UART ima 4 porta: **TX**(Prenos), **RX**(Prijem), **Vcc**(Napon) i **GND**(Tlo). Možda ćete moći da pronađete 4 porta sa slovima **`TX`** i **`RX`** **napisanim** na PCB-u. Ali ako nema indikacija, možda ćete morati sami da ih pronađete koristeći **multimetar** ili **logički analizator**.

Sa **multimetrom** i isključenim uređajem:

* Da biste identifikovali **GND** pin koristite režim **Test kontinuiteta**, postavite zadnju sondi na tlo i testirajte crvenom sondom dok ne čujete zvuk sa multimetra. Na PCB-u se može naći nekoliko GND pinova, pa možda ste pronašli ili niste onaj koji pripada UART-u.
* Da biste identifikovali **VCC port**, postavite **režim DC napona** i postavite ga na 20 V napona. Crna sonda na tlu i crvena sonda na pinu. Uključite uređaj. Ako multimetar meri konstantni napon od 3.3 V ili 5 V, pronašli ste Vcc pin. Ako dobijete druge napon, pokušajte sa drugim portovima.
* Da biste identifikovali **TX** **port**, **režim DC napona** do 20 V napona, crna sonda na tlu, crvena sonda na pinu, i uključite uređaj. Ako primetite da napon fluktuira nekoliko sekundi, a zatim se stabilizuje na vrednosti Vcc, verovatno ste pronašli TX port. To je zato što prilikom uključivanja, šalje neke podatke za debagiranje.
* **RX port** bi trebao biti najbliži od druga 3, ima najmanju fluktuaciju napona i najmanju ukupnu vrednost od svih UART pinova.

Možete da pomešate TX i RX portove i ništa se neće desiti, ali ako pomešate GND i VCC port možete da uništite krug.

Na nekim ciljnim uređajima, UART port je onemogućen od strane proizvođača onemogućavanjem RX ili TX ili čak oba. U tom slučaju, može biti korisno pratiti veze na ploči i pronaći neku tačku prekida. Jak znak o potvrdi neprepoznavanja UART-a i prekida kola je provera garancije uređaja. Ako je uređaj isporučen sa nekom garancijom, proizvođač ostavlja neke debag interfejse (u ovom slučaju, UART) i stoga, mora da je isključio UART i ponovo ga povezao tokom debagiranja. Ove prekidačke pinove možete povezati lemljenjem ili jumper žicama.

### Identifikacija UART Baud Rate

Najlakši način identifikacije ispravnog baud rate-a je da pogledate **izlaz TX pina i pokušate da pročitate podatke**. Ako podaci koje primate nisu čitljivi, pređite na sledeći mogući baud rate dok podaci ne postanu čitljivi. Možete koristiti USB-za-serijski adapter ili višenamenski uređaj poput Bus Pirate-a za ovo, uparen sa pomoćnim skriptom, poput [baudrate.py](https://github.com/devttys0/baudrate/). Najčešći baud rate-ovi su 9600, 38400, 19200, 57600 i 115200.

{% hint style="danger" %}
Važno je napomenuti da u ovom protokolu morate povezati TX jednog uređaja sa RX drugog!
{% endhint %}

## CP210X UART to TTY Adapter

Čip CP210X se koristi u mnogim prototipnim pločama poput NodeMCU (sa esp8266) za serijsku komunikaciju. Ovi adapteri su relativno jeftini i mogu se koristiti za povezivanje sa UART interfejsom cilja. Uređaj ima 5 pinova: 5V, GND, RXD, TXD, 3.3V. Pazite da povežete napon podržan od strane cilja kako biste izbegli bilo kakvu štetu. Na kraju povežite RXD pin adaptera sa TXD cilja i TXD pin adaptera sa RXD cilja.

U slučaju da adapter nije detektovan, proverite da li su drajveri CP210X instalirani na host sistemu. Kada je adapter detektovan i povezan, mogu se koristiti alati poput picocom, minicom ili screen.

Za listanje uređaja povezanih sa Linux/MacOS sistemima:

```
ls /dev/
```

Za osnovnu interakciju sa UART interfejsom, koristite sledeću komandu:

```
picocom /dev/<adapter> --baud <baudrate>
```

Za minicom, koristite sledeću komandu za konfiguraciju:

```
minicom -s
```

Podesite postavke kao što su brzina prenosa i ime uređaja u opciji `Podešavanje serijskog porta`.

Nakon konfiguracije, koristite komandu `minicom` da biste započeli korišćenje UART konzole.

## Bus Pirate

U ovom scenariju ćemo prisluškivati UART komunikaciju Arduina koji šalje sve ispisane poruke programa na serijski monitor.

```bash
# Check the modes
UART>m
1. HiZ
2. 1-WIRE
3. UART
4. I2C
5. SPI
6. 2WIRE
7. 3WIRE
8. KEYB
9. LCD
10. PIC
11. DIO
x. exit(without change)

# Select UART
(1)>3
Set serial port speed: (bps)
1. 300
2. 1200
3. 2400
4. 4800
5. 9600
6. 19200
7. 38400
8. 57600
9. 115200
10. BRG raw value

# Select the speed the communication is occurring on (you BF all this until you find readable things)
# Or you could later use the macro (4) to try to find the speed
(1)>5
Data bits and parity:
1. 8, NONE *default
2. 8, EVEN
3. 8, ODD
4. 9, NONE

# From now on pulse enter for default
(1)>
Stop bits:
1. 1 *default
2. 2
(1)>
Receive polarity:
1. Idle 1 *default
2. Idle 0
(1)>
Select output type:
1. Open drain (H=Hi-Z, L=GND)
2. Normal (H=3.3V, L=GND)

(1)>
Clutch disengaged!!!
To finish setup, start up the power supplies with command 'W'
Ready

# Start
UART>W
POWER SUPPLIES ON
Clutch engaged!!!

# Use macro (2) to read the data of the bus (live monitor)
UART>(2)
Raw UART input
Any key to exit
Escritura inicial completada:
AAA Hi Dreg! AAA
waiting a few secs to repeat....
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
