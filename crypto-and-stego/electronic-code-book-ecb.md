# Electronic Code Book (ECB)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## ECB

(ECB) Electronic Code Book - simetrična šema enkripcije koja **zamenjuje svaki blok čistog teksta** blokom šifrovane poruke. To je **najjednostavnija** šema enkripcije. Glavna ideja je da se čisti tekst podeli na **blokove od N bita** (zavisi od veličine bloka ulaznih podataka, algoritma enkripcije) i zatim da se svaki blok čistog teksta enkriptuje (dekriptuje) koristeći samo ključ.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB\_decryption.svg/601px-ECB\_decryption.svg.png)

Korišćenje ECB ima više sigurnosnih implikacija:

* **Blokovi iz šifrovane poruke mogu biti uklonjeni**
* **Blokovi iz šifrovane poruke mogu biti premesteni**

## Otkrivanje ranjivosti

Zamislite da se prijavljujete na aplikaciju nekoliko puta i **uvek dobijate isti kolačić**. To je zato što je kolačić aplikacije **`<korisničko_ime>|<lozinka>`**.\
Zatim, generišete dva nova korisnika, oba sa **istom dugom lozinkom** i **skoro** **istim** **korisničkim imenom**.\
Otkrivate da su **blokovi od 8B** gde je **informacija o oba korisnika** ista **jednaki**. Tada pretpostavljate da se možda koristi **ECB**.

Kao u sledećem primeru. Primetite kako ova **2 dekodirana kolačića** imaju nekoliko puta blok **`\x23U\xE45K\xCB\x21\xC8`**

```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```

Ovo je zato što su **korisničko ime i lozinka tih kolačića sadržavali više puta slovo "a"** (na primer). **Blokovi** koji su **različiti** su blokovi koji su sadržavali **barem 1 različit karakter** (možda razdelnik "|" ili neka neophodna razlika u korisničkom imenu).

Sada napadač samo treba da otkrije da li je format `<korisničko ime><razdelnik><lozinka>` ili `<lozinka><razdelnik><korisničko ime>`. Da bi to uradio, može jednostavno **generisati nekoliko korisničkih imena** sa **sličnim i dugim korisničkim imenima i lozinkama** dok ne pronađe format i dužinu razdelnika:

| Dužina korisničkog imena: | Dužina lozinke: | Dužina korisničkog imena+lozinke: | Dužina kolačića (nakon dekodiranja): |
| ------------------------- | --------------- | --------------------------------- | ------------------------------------ |
| 2                         | 2               | 4                                 | 8                                    |
| 3                         | 3               | 6                                 | 8                                    |
| 3                         | 4               | 7                                 | 8                                    |
| 4                         | 4               | 8                                 | 16                                   |
| 7                         | 7               | 14                                | 16                                   |

## Iskorišćavanje ranjivosti

### Uklanjanje celih blokova

Znajući format kolačića (`<korisničko ime>|<lozinka>`), kako biste se predstavili kao korisnik `admin`, kreirajte novog korisnika pod imenom `aaaaaaaaadmin` i dobijte kolačić i dekodirajte ga:

```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```

Možemo videti obrazac `\x23U\xE45K\xCB\x21\xC8` koji je prethodno kreiran sa korisničkim imenom koje je sadržalo samo `a`.\
Zatim, možete ukloniti prvi blok od 8B i dobićete validan kolačić za korisničko ime `admin`:

```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```

### Померање блокова

У многим базама података је исто да претражујете `WHERE username='admin';` или `WHERE username='admin ';` _(Обратите пажњу на додатне размаке)_

Дакле, још један начин да се представите као корисник `admin` би био:

* Генеришите корисничко име тако да је `len(<username>) + len(<delimiter) % len(block)`. Са величином блока од `8B` можете генерисати корисничко име под називом: `username` , са разделником `|` делови `<username><delimiter>` ће генерисати 2 блока од 8B.
* Затим, генеришите лозинку која ће попунити тачан број блокова који садрже корисничко име које желимо да се представимо и размаке, на пример: `admin`

Колачић овог корисника ће бити састављен од 3 блока: прва 2 блока су блокови корисничког имена + разделник, а трећи је блок лозинке (која се претвара у корисничко име): `username |admin`

**Затим, само замените први блок са последњим и представљате се као корисник `admin`: `admin |username`**

### Референце

* [http://cryptowiki.net/index.php?title=Electronic\_Code\_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic\_Code\_Book\_\(ECB\))

<details>

<summary><strong>Научите хаковање AWS-а од нуле до хероја са</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Други начини да подржите HackTricks:

* Ако желите да видите **вашу компанију рекламирану на HackTricks** или **преузмете HackTricks у PDF формату** Проверите [**ПРЕТПЛАТНЕ ПЛАНОВЕ**](https://github.com/sponsors/carlospolop)!
* Набавите [**званични PEASS & HackTricks сувенир**](https://peass.creator-spring.com)
* Откријте [**The PEASS Family**](https://opensea.io/collection/the-peass-family), нашу колекцију ексклузивних [**NFT-ова**](https://opensea.io/collection/the-peass-family)
* **Придружите се** 💬 [**Discord групи**](https://discord.gg/hRep4RUj7f) или [**телеграм групи**](https://t.me/peass) или **пратите** нас на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поделите своје хакерске трикове слањем PR-ова на** [**HackTricks**](https://github.com/carlospolop/hacktricks) и [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторијуме.

</details>
