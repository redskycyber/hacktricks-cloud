# Hardware Hacking

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite videti **oglašavanje vaše kompanije na HackTricks-u** ili **preuzeti HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

##

## JTAG

JTAG omogućava izvođenje graničnog skeniranja. Granično skeniranje analizira određenu elektroniku, uključujući ugrađene granične skenirajuće ćelije i registre za svaki pin.

JTAG standard definiše **specifične komande za sprovođenje graničnog skeniranja**, uključujući sledeće:

* **BYPASS** vam omogućava testiranje određenog čipa bez prolaska kroz druge čipove.
* **SAMPLE/PRELOAD** uzima uzorak podataka koji ulaze i izlaze iz uređaja kada je u normalnom režimu rada.
* **EXTEST** postavlja i čita stanja pinova.

Takođe može podržavati i druge komande kao što su:

* **IDCODE** za identifikaciju uređaja
* **INTEST** za internu proveru uređaja

Možete naići na ove instrukcije kada koristite alat poput JTAGulator-a.

### Test pristupni port

Granično skeniranje uključuje testiranje četvoropinske **Test pristupne tačke (TAP)**, opšte namenskog porta koji pruža **pristup JTAG test podršci** funkcija ugrađenih u komponentu. TAP koristi sledećih pet signala:

* Ulaz test sata (**TCK**) TCK je **takt** koji definiše koliko često će TAP kontroler preduzeti jednu radnju (drugim rečima, preći na sledeće stanje u mašini stanja).
* Ulaz za izbor test moda (**TMS**) TMS kontroliše **konačnu stanje mašinu**. Prilikom svakog takta sata, JTAG TAP kontroler uređaja proverava napon na TMS pinu. Ako je napon ispod određenog praga, signal se smatra niskim i tumači se kao 0, dok se ako je napon iznad određenog praga, signal smatra visokim i tumači se kao 1.
* Ulaz test podataka (**TDI**) TDI je pin koji šalje **podatke u čip putem skenirajućih ćelija**. Svaki proizvođač je odgovoran za definisanje protokola komunikacije preko ovog pina, jer JTAG to ne definiše.
* Izlaz test podataka (**TDO**) TDO je pin koji šalje **podatke iz čipa**.
* Ulaz za resetovanje testa (**TRST**) Opcioni TRST resetuje konačnu stanje mašinu **na poznato dobro stanje**. Alternativno, ako se TMS drži na 1 tokom pet uzastopnih ciklusa sata, poziva se reset, na isti način kao što bi to radio TRST pin, zbog čega je TRST opcionalan.

Ponekad ćete moći da pronađete ove pinove obeležene na PCB-u. U drugim slučajevima možda ćete morati da ih **pronađete**.

### Identifikacija JTAG pinova

Najbrži, ali najskuplji način za otkrivanje JTAG portova je korišćenje **JTAGulator-a**, uređaja koji je specifično napravljen za tu svrhu (iako može **takođe otkriti UART pinout-ove**).

Ima **24 kanala** na koje možete povezati pinove ploča. Zatim vrši **BF napad** svih mogućih kombinacija slanjem **IDCODE** i **BYPASS** graničnih skenirajućih komandi. Ako primi odgovor, prikazuje kanal koji odgovara svakom JTAG signalu.

Jeftiniji, ali mnogo sporiji način identifikacije JTAG pinout-ova je korišćenje [**JTAGenum**](https://github.com/cyphunk/JTAGenum/) učitanog na Arduino-kompatibilnom mikrokontroleru.

Koristeći **JTAGenum**, prvo biste **definisali pinove za ispitivanje** uređaja koje ćete koristiti za numeraciju. Morali biste se pozvati na dijagram pinova uređaja, a zatim povezati ove pinove sa test tačkama na ciljnom uređaju.

Treći način za identifikaciju JTAG pinova je **pregledanje PCB-a** u potrazi za jednim od pinout-ova. U nekim slučajevima, PCB-ovi mogu prikladno obezbediti **Tag-Connect interfejs**, što je jasan pokazatelj da ploča ima JTAG konektor. Možete videti kako taj interfejs izgleda na [https://www.tag-connect.com/info/](https://www.tag-connect.com/info/). Dodatno, pregledanje **datasheet-ova čipsetova na PCB-u** može otkriti dijagrame pinout-a koji ukazuju na JTAG interfejse.

## SDW

SWD je ARM-specifični protokol dizajniran za debagovanje.

SWD interfejs zahteva **dva pina**: dvosmerni signal **SWDIO**, koji je ekvivalent JTAG-ovim pinovima **TDI i TDO i takt**, i **SWCLK**, koji je ekvivalent **TCK** u JTAG-u. Mnogi uređaji podržavaju **Serial Wire ili JTAG Debug Port (SWJ-DP)**, kombinovani JTAG i SWD interfejs koji vam omogućava da povežete SWD ili JTAG probu sa ciljem.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite videti **oglašavanje vaše kompanije na HackTricks-u** ili **preuzeti HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
