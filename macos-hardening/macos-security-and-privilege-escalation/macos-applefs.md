# macOS AppleFS

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Apple Proprietary File System (APFS)

**Apple File System (APFS)** je moderni fajl sistem dizajniran da zameni Hierarchical File System Plus (HFS+). Njegov razvoj je pokrenut potrebom za **unapređenom performansom, sigurnošću i efikasnošću**.

Neki značajni elementi APFS-a uključuju:

1. **Deljenje prostora**: APFS omogućava više volumena da **deli isti slobodni prostor** na jednom fizičkom uređaju. Ovo omogućava efikasnije korišćenje prostora jer se volumeni mogu dinamički povećavati i smanjivati bez potrebe za ručnim promenama veličine ili reparticioniranjem.
2. Ovo znači, u poređenju sa tradicionalnim particijama na fajl diskovima, **da u APFS-u različite particije (volumeni) dele sav prostor na disku**, dok je uobičajena particija obično imala fiksnu veličinu.
3. **Snapshotovi**: APFS podržava **kreiranje snapshotova**, koji su **samo za čitanje**, trenutni trenuci fajl sistema. Snapshotovi omogućavaju efikasne rezerve i jednostavna vraćanja sistema, jer zauzimaju minimalan dodatni prostor i mogu se brzo kreirati ili vratiti.
4. **Klonovi**: APFS može **kreirati klonove fajlova ili direktorijuma koji dele isti prostor za skladištenje** kao originalni fajl sve dok se klon ili originalni fajl ne izmene. Ova funkcija pruža efikasan način za kreiranje kopija fajlova ili direktorijuma bez dupliranja prostora za skladištenje.
5. **Enkripcija**: APFS **nativno podržava enkripciju celog diska**, kao i enkripciju po fajlu i po direktorijumu, poboljšavajući sigurnost podataka u različitim slučajevima upotrebe.
6. **Zaštita od pada sistema**: APFS koristi šemu metapodataka **kopiranja pri pisanju koja obezbeđuje doslednost fajl sistema** čak i u slučajevima iznenadnog gubitka napajanja ili pada sistema, smanjujući rizik od oštećenja podataka.

Ukupno gledano, APFS nudi moderniji, fleksibilniji i efikasniji fajl sistem za Apple uređaje, sa fokusom na unapređenu performansu, pouzdanost i sigurnost.

```bash
diskutil list # Get overview of the APFS volumes
```

## Firmlinks

`Data` volumen je montiran u **`/System/Volumes/Data`** (možete to provjeriti sa `diskutil apfs list`).

Lista firmlinks-a se može pronaći u datoteci **`/usr/share/firmlinks`**.

```bash
cat /usr/share/firmlinks
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
[...]
```

Na **levom** delu se nalazi putanja direktorijuma na **Sistemskom volumenu**, a na **desnom** delu se nalazi putanja direktorijuma gde se mapira na **Podatkovnom volumenu**. Dakle, `/library` --> `/system/Volumes/data/library`

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
