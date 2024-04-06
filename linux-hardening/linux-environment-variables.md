# Linux Environment Variables

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Globalne promenljive

Globalne promenljive **će biti** nasleđene od **procesa potomaka**.

Možete kreirati globalnu promenljivu za vašu trenutnu sesiju koristeći:

```bash
export MYGLOBAL="hello world"
echo $MYGLOBAL #Prints: hello world
```

Ova promenljiva će biti dostupna vašim trenutnim sesijama i njihovim podprocesima.

Možete **ukloniti** promenljivu tako što ćete uraditi:

```bash
unset MYGLOBAL
```

## Lokalne promenljive

**Lokalne promenljive** mogu biti **pristupljene** samo od strane **trenutne ljuske/skripte**.

```bash
LOCAL="my local"
echo $LOCAL
unset LOCAL
```

## Lista trenutnih promenljivih

```bash
set
env
printenv
cat /proc/$$/environ
cat /proc/`python -c "import os; print(os.getppid())"`/environ
```

## Zajedničke promenljive

Sa: [https://geek-university.com/linux/common-environment-variables/](https://geek-university.com/linux/common-environment-variables/)

* **DISPLAY** – prikaz koji koristi **X**. Ova promenljiva obično ima vrednost **:0.0**, što znači prvi prikaz na trenutnom računaru.
* **EDITOR** – preferirani tekst editor korisnika.
* **HISTFILESIZE** – maksimalan broj linija sadržanih u datoteci istorije.
* **HISTSIZE** – Broj linija dodatih u datoteku istorije kada korisnik završi sesiju.
* **HOME** – vaš matični direktorijum.
* **HOSTNAME** – ime računara.
* **LANG** – vaš trenutni jezik.
* **MAIL** – lokacija poštanskog sandučeta korisnika. Obično **/var/spool/mail/USER**.
* **MANPATH** – lista direktorijuma u kojima se traže stranice priručnika.
* **OSTYPE** – tip operativnog sistema.
* **PS1** – podrazumevani znak za unos u bash-u.
* **PATH** – čuva putanje svih direktorijuma koji sadrže binarne datoteke koje želite da izvršite samo navođenjem imena datoteke, a ne relativne ili apsolutne putanje.
* **PWD** – trenutni radni direktorijum.
* **SHELL** – putanja do trenutne komandne ljuske (na primer, **/bin/bash**).
* **TERM** – trenutni tip terminala (na primer, **xterm**).
* **TZ** – vaša vremenska zona.
* **USER** – vaše trenutno korisničko ime.

## Interesantne promenljive za hakovanje

### **HISTFILESIZE**

Promenite **vrednost ove promenljive na 0**, tako da kada **završite sesiju**, datoteka istorije (\~/.bash\_history) **će biti obrisana**.

```bash
export HISTFILESIZE=0
```

### **HISTSIZE**

Promenite **vrednost ove promenljive na 0**, tako da kada **završite sesiju**, bilo koja komanda neće biti dodata u **istorijski fajl** (\~/.bash\_history).

```bash
export HISTSIZE=0
```

### http\_proxy & https\_proxy

Procesi će koristiti **proxy** koji je ovde naveden da bi se povezali na internet preko **http ili https**.

```bash
export http_proxy="http://10.10.10.10:8080"
export https_proxy="http://10.10.10.10:8080"
```

### SSL\_CERT\_FILE & SSL\_CERT\_DIR

Procesi će verovati sertifikatima navedenim u **ovim env varijablama**.

```bash
export SSL_CERT_FILE=/path/to/ca-bundle.pem
export SSL_CERT_DIR=/path/to/ca-certificates
```

### PS1

Promenite izgled vašeg prompta.

[**Ovo je primer**](https://gist.github.com/carlospolop/43f7cd50f3deea972439af3222b68808)

Root:

![](<../.gitbook/assets/image (87).png>)

Običan korisnik:

![](<../.gitbook/assets/image (88).png>)

Jedan, dva i tri pozadinski poslovi:

![](<../.gitbook/assets/image (89).png>)

Jedan pozadinski posao, jedan zaustavljen i poslednja komanda nije završena ispravno:

![](<../.gitbook/assets/image (90).png>)

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
