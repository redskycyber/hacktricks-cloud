# Crypto CTFs Tricks

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Online baze heševa

* _**Pretražite na Google-u**_
* [http://hashtoolkit.com/reverse-hash?hash=4d186321c1a7f0f354b297e8914ab240](http://hashtoolkit.com/reverse-hash?hash=4d186321c1a7f0f354b297e8914ab240)
* [https://www.onlinehashcrack.com/](https://www.onlinehashcrack.com)
* [https://crackstation.net/](https://crackstation.net)
* [https://md5decrypt.net/](https://md5decrypt.net)
* [https://www.onlinehashcrack.com](https://www.onlinehashcrack.com)
* [https://gpuhash.me/](https://gpuhash.me)
* [https://hashes.org/search.php](https://hashes.org/search.php)
* [https://www.cmd5.org/](https://www.cmd5.org)
* [https://hashkiller.co.uk/Cracker/MD5](https://hashkiller.co.uk/Cracker/MD5)
* [https://www.md5online.org/md5-decrypt.html](https://www.md5online.org/md5-decrypt.html)

## Čarobni automatski rešavači

* [**https://github.com/Ciphey/Ciphey**](https://github.com/Ciphey/Ciphey)
* [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/) (Magic modul)
* [https://github.com/dhondta/python-codext](https://github.com/dhondta/python-codext)
* [https://www.boxentriq.com/code-breaking](https://www.boxentriq.com/code-breaking)

## Enkoderi

Većina enkodiranih podataka može se dekodirati pomoću ovih 2 resursa:

* [https://www.dcode.fr/tools-list](https://www.dcode.fr/tools-list)
* [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)

### Automatski rešavači za zamenu

* [https://www.boxentriq.com/code-breaking/cryptogram](https://www.boxentriq.com/code-breaking/cryptogram)
* [https://quipqiup.com/](https://quipqiup.com) - Veoma dobro!

#### Cezar - ROTx automatski rešavači

* [https://www.nayuki.io/page/automatic-caesar-cipher-breaker-javascript](https://www.nayuki.io/page/automatic-caesar-cipher-breaker-javascript)

#### Atbash šifra

* [http://rumkin.com/tools/cipher/atbash.php](http://rumkin.com/tools/cipher/atbash.php)

### Automatski rešavači za osnovne enkodiranje

Proverite sve ove baze sa: [https://github.com/dhondta/python-codext](https://github.com/dhondta/python-codext)

* **Ascii85**
* `BQ%]q@psCd@rH0l`
* **Base26** \[_A-Z_]
* `BQEKGAHRJKHQMVZGKUXNT`
* **Base32** \[_A-Z2-7=_]
* `NBXWYYLDMFZGCY3PNRQQ====`
* **Zbase32** \[_ybndrfg8ejkmcpqxot1uwisza345h769_]
* `pbzsaamdcf3gna5xptoo====`
* **Base32 Geohash** \[_0-9b-hjkmnp-z_]
* `e1rqssc3d5t62svgejhh====`
* **Base32 Crockford** \[_0-9A-HJKMNP-TV-Z_]
* `D1QPRRB3C5S62RVFDHGG====`
* **Base32 Extended Hexadecimal** \[_0-9A-V_]
* `D1NMOOB3C5P62ORFDHGG====`
* **Base45** \[_0-9A-Z $%\*+-./:_]
* `59DPVDGPCVKEUPCPVD`
* **Base58 (bitcoin)** \[_1-9A-HJ-NP-Za-km-z_]
* `2yJiRg5BF9gmsU6AC`
* **Base58 (flickr)** \[_1-9a-km-zA-HJ-NP-Z_]
* `2YiHqF5bf9FLSt6ac`
* **Base58 (ripple)** \[_rpshnaf39wBUDNEGHJKLM4PQ-T7V-Z2b-eCg65jkm8oFqi1tuvAxyz_]
* `pyJ5RgnBE9gm17awU`
* **Base62** \[_0-9A-Za-z_]
* `g2AextRZpBKRBzQ9`
* **Base64** \[_A-Za-z0-9+/=_]
* `aG9sYWNhcmFjb2xh`
* **Base67** \[_A-Za-z0-9-_.!\~\_]
* `NI9JKX0cSUdqhr!p`
* **Base85 (Ascii85)** \[_!"#$%&'()\*+,-./0-9:;<=>?@A-Z\[\\]^\_\`a-u_]
* `BQ%]q@psCd@rH0l`
* **Base85 (Adobe)** \[_!"#$%&'()\*+,-./0-9:;<=>?@A-Z\[\\]^\_\`a-u_]
* `<~BQ%]q@psCd@rH0l~>`
* **Base85 (IPv6 or RFC1924)** \[_0-9A-Za-z!#$%&()\*+-;<=>?@^_\`{|}\~\_]
* `Xm4y`V\_|Y(V{dF>\`
* **Base85 (xbtoa)** \[_!"#$%&'()\*+,-./0-9:;<=>?@A-Z\[\\]^\_\`a-u_]
* `xbtoa Begin\nBQ%]q@psCd@rH0l\nxbtoa End N 12 c E 1a S 4e6 R 6991d`
* **Base85 (XML)** \[_0-9A-Za-y!#$()\*+,-./:;=?@^\`{|}\~z\__]
* `Xm4y|V{~Y+V}dF?`
* **Base91** \[_A-Za-z0-9!#$%&()\*+,./:;<=>?@\[]^\_\`{|}\~"_]
* `frDg[*jNN!7&BQM`
*
* `DmPsv8J7qrlKEoY7`
* **MEGAN35** \[_3G-Ub=c-pW-Z/12+406-9Vaq-zA-F5_]
* `kLD8iwKsigSalLJ5`
* **ZONG22** \[_ZKj9n+yf0wDVX1s/5YbdxSo=ILaUpPBCHg8uvNO4klm6iJGhQ7eFrWczAMEq3RTt2_]
* `ayRiIo1gpO+uUc7g`
* **ESAB46** \[]
* `3sHcL2NR8WrT7mhR`
* **MEGAN45** \[]
* `kLD8igSXm2KZlwrX`
* **TIGO3FX** \[]
* `7AP9mIzdmltYmIP9mWXX`
* **TRIPO5** \[]
* `UE9vSbnBW6psVzxB`
* **FERON74** \[]
* `PbGkNudxCzaKBm0x`
* **GILA7** \[]
* `D+nkv8C1qIKMErY1`
* **Citrix CTX1** \[]
* `MNGIKCAHMOGLKPAKMMGJKNAINPHKLOBLNNHILCBHNOHLLPBK`

[http://k4.cba.pl/dw/crypo/tools/eng\_atom128c.html](http://k4.cba.pl/dw/crypo/tools/eng\_atom128c.html) - 404 Dead: [https://web.archive.org/web/20190228181208/http://k4.cba.pl/dw/crypo/tools/eng\_hackerize.html](https://web.archive.org/web/20190228181208/http://k4.cba.pl/dw/crypo/tools/eng\_hackerize.html)

### HackerizeXS \[_╫Λ↻├☰┏_]

```
╫☐↑Λ↻Λ┏Λ↻☐↑Λ
```

* [http://k4.cba.pl/dw/crypo/tools/eng\_hackerize.html](http://k4.cba.pl/dw/crypo/tools/eng\_hackerize.html) - 404 Dead: [https://web.archive.org/web/20190228181208/http://k4.cba.pl/dw/crypo/tools/eng\_hackerize.html](https://web.archive.org/web/20190228181208/http://k4.cba.pl/dw/crypo/tools/eng\_hackerize.html)

### Morse

### Морзе

```
.... --- .-.. -.-. .- .-. .- -.-. --- .-.. .-
```

* [http://k4.cba.pl/dw/crypo/tools/eng\_morse-encode.html](http://k4.cba.pl/dw/crypo/tools/eng\_morse-encode.html) - 404 Dead: [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)

### UUencoder

UUencoder je tehnika enkodiranja koja se koristi za pretvaranje binarnih podataka u ASCII format. Ova tehnika je često korišćena u kriptografiji i komunikaciji putem mejla. UUencoder konvertuje binarne podatke u ASCII karaktere kako bi se omogućilo slanje podataka putem tekstualnih formata koji podržavaju samo ASCII karaktere. Ova tehnika je korisna u situacijama kada je potrebno preneti binarne podatke preko kanala koji podržava samo tekstualne formate.

```
begin 644 webutils_pl
M2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(
M3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/
F3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$%(3TQ!2$],04A/3$$`
`
end
```

* [http://www.webutils.pl/index.php?idx=uu](http://www.webutils.pl/index.php?idx=uu)

### XXEncoder

XXEncoder je jednostavan alat koji se koristi za enkodiranje i dekodiranje podataka u različitim formatima. Može se koristiti za enkodiranje tekstualnih podataka u format koji je pogodan za prenos preko HTTP zahteva. Takođe može biti koristan za enkodiranje binarnih podataka kao što su slike ili fajlovi. Ovaj alat je često korišćen u CTF takmičenjima za rešavanje kriptografskih izazova.

```
begin 644 webutils_pl
hG2xAEIVDH236Hol-G2xAEIVDH236Hol-G2xAEIVDH236Hol-G2xAEIVDH236
5Hol-G2xAEE++
end
```

* [www.webutils.pl/index.php?idx=xx](https://github.com/carlospolop/hacktricks/tree/bf578e4c5a955b4f6cdbe67eb4a543e16a3f848d/crypto/www.webutils.pl/index.php?idx=xx)

### YEncoder

YEncoder je jednostavan algoritam enkodiranja koji se koristi za enkodiranje binarnih podataka u ASCII format. Ovaj algoritam je često korišćen u CTF takmičenjima za skrivanje informacija.

Da biste dekodovali YEncoded podatke, možete koristiti različite online alate ili napisati sopstveni skript. Kada dekodujete podatke, dobićete originalne binarne podatke.

Evo primera kako možete dekodovati YEncoded podatke koristeći Python skript:

```python
import binascii

def ydecode(data):
    decoded_data = ''
    i = 0
    while i < len(data):
        if data[i] == '=':
            break
        char = ord(data[i]) - 42
        decoded_data += chr(char)
        i += 1
    return binascii.unhexlify(decoded_data)

encoded_data = 'Y29vbCBkYXRh'
decoded_data = ydecode(encoded_data)
print(decoded_data)
```

Ovaj skript će dekodovati YEncoded podatke `'Y29vbCBkYXRh'` i ispisati originalne binarne podatke `'cool data'`.

Kada se susretnete sa YEncoded podacima tokom CTF takmičenja, važno je da pažljivo proučite kako su podaci enkodirani i koristite odgovarajuće metode za dekodiranje.

```
=ybegin line=128 size=28 name=webutils_pl
ryvkryvkryvkryvkryvkryvkryvk
=yend size=28 crc32=35834c86
```

* [http://www.webutils.pl/index.php?idx=yenc](http://www.webutils.pl/index.php?idx=yenc)

### BinHex

BinHex je metoda enkodiranja koja se koristi za pretvaranje binarnih podataka u tekstualni format. Ova tehnika je često korišćena u prošlosti za prenos binarnih fajlova putem emaila ili drugih tekstualnih kanala komunikacije. BinHex enkodiranje koristi 6-bitnu ASCII enkodiranu verziju originalnih binarnih podataka. Ova metoda takođe dodaje kontrolne karaktere kako bi se obezbedila integritet podataka prilikom prenosa. BinHex enkodiranje se može dekodirati kako bi se dobili originalni binarni podaci.

```
(This file must be converted with BinHex 4.0)
:#hGPBR9dD@acAh"X!$mr2cmr2cmr!!!!!!!8!!!!!-ka5%p-38K26%&)6da"5%p
-38K26%'d9J!!:
```

* [http://www.webutils.pl/index.php?idx=binhex](http://www.webutils.pl/index.php?idx=binhex)

### ASCII85

ASCII85 je metoda enkodiranja koja se koristi za pretvaranje binarnih podataka u tekstualni format. Ova tehnika je često korišćena u CTF izazovima i može biti korisna prilikom rešavanja kriptografskih zagonetki. ASCII85 enkodiranje se zasniva na pretvaranju svakih 4 bajta binarnih podataka u 5 ASCII karaktera. Ova tehnika je slična Base64 enkodiranju, ali ASCII85 omogućava efikasnije enkodiranje podataka. Da biste dekodovali ASCII85 enkodirane podatke, možete koristiti različite alate i biblioteke dostupne na internetu.

```
<~85DoF85DoF85DoF85DoF85DoF85DoF~>
```

* [http://www.webutils.pl/index.php?idx=ascii85](http://www.webutils.pl/index.php?idx=ascii85)

### Dvorak tastatura

```
drnajapajrna
```

* [https://www.geocachingtoolbox.com/index.php?lang=sr\&page=dvorakKeyboard](https://www.geocachingtoolbox.com/index.php?lang=sr\&page=dvorakKeyboard)

### A1Z26

Slova u njihovu numeričku vrednost

```
8 15 12 1 3 1 18 1 3 15 12 1
```

### Enkodiranje pomoću Afine šifre

Slovo u broj `(ax+b)%26` (_a_ i _b_ su ključevi, a _x_ je slovo) i rezultat se vraća nazad u slovo

```
krodfdudfrod
```

### SMS Kod

**Multitap** [zamenjuje slovo](https://www.dcode.fr/word-letter-change) ponavljajućim brojevima definisanim odgovarajućim kodom na tastaturi mobilnog [telefona](https://www.dcode.fr/phone-keypad-cipher) (Ovaj režim se koristi prilikom pisanja SMS poruka).\
Na primer: 2=A, 22=B, 222=C, 3=D...\
Možete prepoznati ovaj kod jer ćete videti\*\* više puta ponovljene brojeve\*\*.

Ovaj kod možete dekodirati na: [https://www.dcode.fr/multitap-abc-cipher](https://www.dcode.fr/multitap-abc-cipher)

### Bacon Kod

Zamenite svako slovo sa 4 A ili B (ili 1 i 0)

```
00111 01101 01010 00000 00010 00000 10000 00000 00010 01101 01010 00000
AABBB ABBAB ABABA AAAAA AAABA AAAAA BAAAA AAAAA AAABA ABBAB ABABA AAAAA
```

### Runovi

![](../.gitbook/assets/runes.jpg)

## Kompresija

**Raw Deflate** i **Raw Inflate** (obe možete pronaći u Cyberchef-u) mogu kompresovati i dekompresovati podatke bez zaglavlja.

## Jednostavna Kriptografija

### XOR - Automatsko rešavanje

* [https://wiremask.eu/tools/xor-cracker/](https://wiremask.eu/tools/xor-cracker/)

### Bifid

Potreban je ključna reč

```
fgaargaamnlunesuneoa
```

### Vigenere

Potreban je ključna reč

```
wodsyoidrods
```

* [https://www.guballa.de/vigenere-solver](https://www.guballa.de/vigenere-solver)
* [https://www.dcode.fr/vigenere-cipher](https://www.dcode.fr/vigenere-cipher)
* [https://www.mygeocachingprofile.com/codebreaker.vigenerecipher.aspx](https://www.mygeocachingprofile.com/codebreaker.vigenerecipher.aspx)

## Jaka kriptografija

### Fernet

2 base64 stringa (token i ključ)

```
Token:
gAAAAABWC9P7-9RsxTz_dwxh9-O2VUB7Ih8UCQL1_Zk4suxnkCvb26Ie4i8HSUJ4caHZuiNtjLl3qfmCv_fS3_VpjL7HxCz7_Q==

Key:
-s6eI5hyNh8liH7Gq0urPC-vzPgNnxauKvRO4g03oYI=
```

* [https://asecuritysite.com/encryption/ferdecode](https://asecuritysite.com/encryption/ferdecode)

### Samirovo deljenje tajni

Tajna se deli na X delova, a za njeno povraćanje potrebno je Y delova (_Y <=X_).

```
8019f8fa5879aa3e07858d08308dc1a8b45
80223035713295bddf0b0bd1b10a5340b89
803bc8cf294b3f83d88e86d9818792e80cd
```

[http://christian.gen.co/secrets/](http://christian.gen.co/secrets/)

### OpenSSL brute-force

* [https://github.com/glv2/bruteforce-salted-openssl](https://github.com/glv2/bruteforce-salted-openssl)
* [https://github.com/carlospolop/easy\_BFopensslCTF](https://github.com/carlospolop/easy\_BFopensslCTF)

## Alati

* [https://github.com/Ganapati/RsaCtfTool](https://github.com/Ganapati/RsaCtfTool)
* [https://github.com/lockedbyte/cryptovenom](https://github.com/lockedbyte/cryptovenom)
* [https://github.com/nccgroup/featherduster](https://github.com/nccgroup/featherduster)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
