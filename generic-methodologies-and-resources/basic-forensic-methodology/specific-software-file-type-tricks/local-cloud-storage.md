# Local Cloud Storage

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodičnu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako izgradite i **automatizujete radne tokove** pokretane najnaprednijim alatima zajednice.\
Dobijte pristup danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## OneDrive

U Windows-u, možete pronaći OneDrive fasciklu u `\Users\<korisničko_ime>\AppData\Local\Microsoft\OneDrive`. A unutar `logs\Personal` moguće je pronaći fajl `SyncDiagnostics.log` koji sadrži neke zanimljive podatke u vezi sa sinhronizovanim fajlovima:

* Veličina u bajtovima
* Datum kreiranja
* Datum modifikacije
* Broj fajlova u oblaku
* Broj fajlova u fascikli
* **CID**: Jedinstveni ID OneDrive korisnika
* Vreme generisanja izveštaja
* Veličina HD-a OS-a

Kada pronađete CID, preporučljivo je **pretražiti fajlove koji sadrže ovaj ID**. Možda ćete moći pronaći fajlove sa imenima: _**\<CID>.ini**_ i _**\<CID>.dat**_ koji mogu sadržati zanimljive informacije poput imena fajlova sinhronizovanih sa OneDrive-om.

## Google Drive

U Windows-u, možete pronaći glavnu Google Drive fasciklu u `\Users\<korisničko_ime>\AppData\Local\Google\Drive\user_default`\
Ova fascikla sadrži fajl nazvan Sync\_log.log sa informacijama poput email adrese naloga, imena fajlova, vremenskih oznaka, MD5 heševa fajlova, itd. Čak i obrisani fajlovi se pojavljuju u tom log fajlu sa odgovarajućim MD5 vrednostima.

Fajl **`Cloud_graph\Cloud_graph.db`** je sqlite baza podataka koja sadrži tabelu **`cloud_graph_entry`**. U ovoj tabeli možete pronaći **ime** **sinhronizovanih** **fajlova**, vreme modifikacije, veličinu i MD5 kontrolnu sumu fajlova.

Podaci tabele baze podataka **`Sync_config.db`** sadrže email adresu naloga, putanje deljenih fascikli i verziju Google Drive-a.

## Dropbox

Dropbox koristi **SQLite baze podataka** za upravljanje fajlovima. U ovim\
Možete pronaći baze podataka u fasciklama:

* `\Users\<korisničko_ime>\AppData\Local\Dropbox`
* `\Users\<korisničko_ime>\AppData\Local\Dropbox\Instance1`
* `\Users\<korisničko_ime>\AppData\Roaming\Dropbox`

Glavne baze podataka su:

* Sigstore.dbx
* Filecache.dbx
* Deleted.dbx
* Config.dbx

Ekstenzija ".dbx" znači da su **baze podataka enkriptovane**. Dropbox koristi **DPAPI** ([https://docs.microsoft.com/en-us/previous-versions/ms995355(v=msdn.10)?redirectedfrom=MSDN](https://docs.microsoft.com/en-us/previous-versions/ms995355\(v=msdn.10\)?redirectedfrom=MSDN))

Da biste bolje razumeli enkripciju koju Dropbox koristi, možete pročitati [https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html](https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html).

Međutim, glavne informacije su:

* **Entropija**: d114a55212655f74bd772e37e64aee9b
* **So**: 0D638C092E8B82FC452883F95F355B8E
* **Algoritam**: PBKDF2
* **Iteracije**: 1066

Osim tih informacija, da biste dešifrovali baze podataka, još uvek vam je potrebno:

* **Enkriptovani DPAPI ključ**: Možete ga pronaći u registru unutar `NTUSER.DAT\Software\Dropbox\ks\client` (izvezite ove podatke kao binarne)
* **`SYSTEM`** i **`SECURITY`** košnice
* **DPAPI master ključevi**: Koji se mogu pronaći u `\Users\<korisničko_ime>\AppData\Roaming\Microsoft\Protect`
* **korisničko ime** i **šifra** Windows korisnika

Zatim možete koristiti alat [**DataProtectionDecryptor**](https://nirsoft.net/utils/dpapi\_data\_decryptor.html)**:**

![](<../../../.gitbook/assets/image (448).png>)

Ako sve ide kako se očekuje, alat će pokazati **primarni ključ** koji vam je potreban da biste **koristili za oporavak originalnog**. Da biste oporavili originalni, jednostavno koristite ovaj [cyber\_chef recept](https://gchq.github.io/CyberChef/#recipe=Derive\_PBKDF2\_key\(%7B'option':'Hex','string':'98FD6A76ECB87DE8DAB4623123402167'%7D,128,1066,'SHA1',%7B'option':'Hex','string':'0D638C092E8B82FC452883F95F355B8E'%7D\)) stavljajući primarni ključ kao "lozinku" unutar recepta.

Rezultujući heksadecimalni broj je konačni ključ koji se koristi za enkripciju baza podataka koje se mogu dešifrovati sa:

```bash
sqlite -k <Obtained Key> config.dbx ".backup config.db" #This decompress the config.dbx and creates a clear text backup in config.db
```

Baza podataka **`config.dbx`** sadrži:

* **Email**: Email korisnika
* **usernamedisplayname**: Ime korisnika
* **dropbox\_path**: Putanja gde se nalazi Dropbox folder
* **Host\_id: Hash** korišćen za autentifikaciju na oblaku. Ovo se može opozvati samo preko veba.
* **Root\_ns**: Identifikator korisnika

Baza podataka **`filecache.db`** sadrži informacije o svim fajlovima i fasciklama sinhronizovanim sa Dropbox-om. Tabela `File_journal` je ona sa najkorisnijim informacijama:

* **Server\_path**: Putanja gde se fajl nalazi unutar servera (ova putanja je prethodjena `host_id`-om klijenta).
* **local\_sjid**: Verzija fajla
* **local\_mtime**: Datum modifikacije
* **local\_ctime**: Datum kreiranja

Druge tabele unutar ove baze podataka sadrže još interesantnih informacija:

* **block\_cache**: heš svih fajlova i fascikli Dropbox-a
* **block\_ref**: Povezan je ID heša iz tabele `block_cache` sa ID fajla u tabeli `file_journal`
* **mount\_table**: Deljeni folderi Dropbox-a
* **deleted\_fields**: Obrisani fajlovi sa Dropbox-a
* **date\_added**

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) da lako kreirate i **automatizujete radne tokove** uz pomoć najnaprednijih alata zajednice na svetu.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
