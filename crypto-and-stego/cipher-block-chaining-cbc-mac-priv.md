# Cipher Block Chaining CBC-MAC

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## CBC

Ako je **kolačić** samo **korisničko ime** (ili prvi deo kolačića je korisničko ime) i želite da se predstavite kao korisničko ime "**admin**". Tada možete kreirati korisničko ime **"bdmin"** i **bruteforce**-ovati **prvi bajt** kolačića.

## CBC-MAC

**Cipher block chaining message authentication code** (**CBC-MAC**) je metoda koja se koristi u kriptografiji. Radi tako što uzima poruku i enkriptuje je blok po blok, pri čemu je enkripcija svakog bloka povezana sa prethodnim. Ovaj proces stvara **lanac blokova**, osiguravajući da čak i promena jednog jedinog bita originalne poruke dovede do nepredvidive promene poslednjeg bloka enkriptovanih podataka. Da bi se izvršila ili poništila takva promena, potreban je ključ za enkripciju, što osigurava bezbednost.

Da bi se izračunao CBC-MAC poruke m, poruka m se enkriptuje u CBC režimu sa nulom kao inicijalizacionim vektorom i čuva se poslednji blok. Sledeća slika prikazuje izračunavanje CBC-MAC poruke koja se sastoji od blokova![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) koristeći tajni ključ k i blok šifre E:

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_(en).svg/570px-CBC-MAC\_structure\_(en).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

## Ranjivost

Sa CBC-MAC-om, obično se koristi **IV vrednost 0**.\
Ovo je problem jer dve poznate poruke (`m1` i `m2`) nezavisno generišu dva potpisa (`s1` i `s2`). Dakle:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Zatim, poruka koja se sastoji od konkatenacije m1 i m2 (m3) generiše dva potpisa (s31 i s32):

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Što je moguće izračunati bez poznavanja ključa enkripcije.**

Zamislite da šifrujete ime **Administrator** u blokovima od **8 bajtova**:

* `Administ`
* `rator\00\00\00`

Možete kreirati korisničko ime koje se zove **Administ** (m1) i dobiti potpis (s1).\
Zatim, možete kreirati korisničko ime koje se zove rezultat `rator\00\00\00 XOR s1`. Ovo će generisati `E(m2 XOR s1 XOR 0)` koji je s32.\
Sada možete koristiti s32 kao potpis za puno ime **Administrator**.

#### Rezime

1. Dobijte potpis korisničkog imena **Administ** (m1) koji je s1
2. Dobijte potpis korisničkog imena **rator\x00\x00\x00 XOR s1 XOR 0** koji je s32\*\*.\*\*
3. Postavite kolačić na s32 i to će biti validan kolačić za korisnika **Administrator**.

## Kontrolisanje napada IV

Ako možete kontrolisati korišćeni IV, napad može biti veoma jednostavan.\
Ako su kolačići samo šifrovano korisničko ime, da biste se predstavili kao korisnik "**administrator**" možete kreirati korisnika "**Administrator**" i dobiti njegov kolačić.\
Sada, ako možete kontrolisati IV, možete promeniti prvi bajt IV-a tako da **IV\[0] XOR "A" == IV'\[0] XOR "a"** i ponovo generisati kolačić za korisnika **Administrator**. Ovaj kolačić će biti validan za **predstavljanje** korisnika **administrator** sa početnim **IV**.

### Reference

Više informacija na [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
