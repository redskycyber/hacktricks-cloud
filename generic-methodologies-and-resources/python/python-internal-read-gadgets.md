# Python Internal Read Gadgets

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovne informacije

Različite ranjivosti kao što su [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) ili [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) mogu vam omogućiti **čitanje internih podataka Python-a, ali ne i izvršavanje koda**. Stoga, pentester će morati da iskoristi ova dozvola za čitanje kako bi **dobio osetljive privilegije i eskalirao ranjivost**.

### Flask - Čitanje tajnog ključa

Glavna stranica Flask aplikacije verovatno će imati **`app`** globalni objekat gde je ovaj **tajni ključ konfigurisan**.

```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```

U ovom slučaju moguće je pristupiti ovom objektu koristeći bilo koji uređaj za **pristup globalnim objektima** sa [stranice **Bypass Python sandboxes**](bypass-python-sandboxes/).

U slučaju kada **ranjivost postoji u drugom Python fajlu**, potreban vam je uređaj za pretraživanje fajlova kako biste došli do glavnog fajla i **pristupili globalnom objektu `app.secret_key`** kako biste promenili Flask tajni ključ i bili u mogućnosti da [**povećate privilegije** znajući ovaj ključ](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Payload poput ovog [iz ovog writeup-a](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Koristite ovaj payload da biste **promenili `app.secret_key`** (naziv u vašoj aplikaciji može biti drugačiji) kako biste mogli da potpišete nove i privilegovane flask kolačiće.

### Werkzeug - machine\_id i node uuid

[**Koristeći ove payloade iz ovog writeupa**](https://vozec.fr/writeups/tweedle-dum-dee/) moći ćete da pristupite **machine\_id** i **uuid** čvoru, koji su **glavne tajne** koje vam trebaju za [**generisanje Werkzeug pin-a**](../../network-services-pentesting/pentesting-web/werkzeug.md) koji možete koristiti za pristup python konzoli u `/console` ako je **debug mod omogućen:**

```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```

{% hint style="warning" %}
Imajte na umu da možete dobiti **lokalnu putanju servera do `app.py`** generisanjem neke **greške** na veb stranici koja će vam **prikazati putanju**.
{% endhint %}

Ako je ranjivost u drugom Python fajlu, proverite prethodni trik za pristup objektima iz glavnog Python fajla.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
