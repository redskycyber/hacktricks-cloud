# Bootloader testing

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Preporučeni koraci za modifikaciju konfiguracija pokretanja uređaja i bootloadera poput U-boot-a:

1. **Pristupanje interpretativnom okruženju bootloadera**:

* Tokom pokretanja, pritisnite "0", razmak ili druge identifikovane "magične kodove" kako biste pristupili interpretativnom okruženju bootloadera.

2. **Modifikacija pokretačkih argumenata**:

* Izvršite sledeće komande da biste dodali '`init=/bin/sh`' na pokretačke argumente, omogućavajući izvršavanje shell komande: %%% #printenv #setenv bootargs=console=ttyS0,115200 mem=63M root=/dev/mtdblock3 mtdparts=sflash: rootfstype= hasEeprom=0 5srst=0 init=/bin/sh #saveenv #boot %%%

3. **Podešavanje TFTP servera**:

* Konfigurišite TFTP server za učitavanje slika preko lokalne mreže: %%% #setenv ipaddr 192.168.2.2 #lokalna IP adresa uređaja #setenv serverip 192.168.2.1 #IP adresa TFTP servera #saveenv #reset #ping 192.168.2.1 #provera pristupa mreži #tftp ${loadaddr} uImage-3.6.35 #loadaddr uzima adresu za učitavanje datoteke i ime datoteke slike na TFTP serveru %%%

4. **Korišćenje `ubootwrite.py`**:

* Koristite `ubootwrite.py` da biste napisali U-boot sliku i poslali modifikovani firmware kako biste dobili root pristup.

5. **Provera debug funkcionalnosti**:

* Proverite da li su omogućene debug funkcionalnosti poput detaljnog beleženja, učitavanja proizvoljnih jezgara ili pokretanja sa nepouzdanih izvora.

6. **Oprez pri fizičkom mešanju sa hardverom**:

* Budite oprezni prilikom povezivanja jednog pina sa zemljom i interakcije sa SPI ili NAND flash čipovima tokom sekvence pokretanja uređaja, posebno pre dekompresije jezgra. Konsultujte tehnički list NAND flash čipa pre kratkog spoja pinova.

7. **Konfiguracija lažnog DHCP servera**:

* Postavite lažni DHCP server sa zlonamernim parametrima koje uređaj treba da prihvati tokom PXE pokretanja. Koristite alate poput Metasploit-ovog (MSF) DHCP pomoćnog servera. Modifikujte parametar 'FILENAME' sa komandama za ubrizgavanje komandi kao što su `'a";/bin/sh;#'` da biste testirali validaciju unosa za postupke pokretanja uređaja.

**Napomena**: Koraci koji uključuju fizičku interakciju sa pinovima uređaja (\*označeni zvezdicom) treba da se pristupe sa izuzetnom pažnjom kako bi se izbegla oštećenja uređaja.

### Reference

* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE PRETPLATE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
