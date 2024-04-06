# Writable Sys Path +Dll Hijacking Privesc

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Uvod

Ako ste otkrili da možete **pisati u folderu System Path** (imajte na umu da ovo neće raditi ako možete pisati u folderu User Path), moguće je da možete **povećati privilegije** u sistemu.

Da biste to uradili, možete iskoristiti **Dll Hijacking** gde ćete **preuzeti kontrolu nad bibliotekom koju učitava** servis ili proces sa **većim privilegijama** od vaših, i zato što taj servis učitava Dll koji verovatno ne postoji u celom sistemu, pokušaće da ga učita iz System Path-a gde možete pisati.

Za više informacija o **šta je Dll Hijacking** proverite:

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Povećanje privilegija pomoću Dll Hijacking-a

### Pronalaženje nedostajućeg Dll-a

Prva stvar koju trebate uraditi je da **identifikujete proces** koji se izvršava sa **većim privilegijama** od vas, a koji pokušava **učitati Dll iz System Path-a** u koji možete pisati.

Problem u ovim slučajevima je što su ti procesi verovatno već pokrenuti. Da biste pronašli koji Dll-ovi nedostaju servisima, trebate pokrenuti procmon što je pre moguće (pre nego što se procesi učitaju). Dakle, da biste pronašli nedostajuće .dll-ove, uradite sledeće:

* **Kreirajte** folder `C:\privesc_hijacking` i dodajte putanju `C:\privesc_hijacking` u **System Path env promenljivu**. To možete uraditi **ručno** ili sa **PS**:

```powershell
# Set the folder path to create and check events for
$folderPath = "C:\privesc_hijacking"

# Create the folder if it does not exist
if (!(Test-Path $folderPath -PathType Container)) {
New-Item -ItemType Directory -Path $folderPath | Out-Null
}

# Set the folder path in the System environment variable PATH
$envPath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
if ($envPath -notlike "*$folderPath*") {
$newPath = "$envPath;$folderPath"
[Environment]::SetEnvironmentVariable("PATH", $newPath, "Machine")
}
```

* Pokrenite **`procmon`** i idite na **`Options`** --> **`Enable boot logging`** i pritisnite **`OK`** u prozoru za potvrdu.
* Zatim, **ponovo pokrenite** računar. Kada se računar ponovo pokrene, **`procmon`** će početi **snimanje** događaja odmah.
* Kada se **Windows** pokrene, ponovo pokrenite **`procmon`**, reći će vam da je već pokrenut i pitati vas da li želite da sačuvate događaje u datoteci. Recite **da** i **sačuvajte događaje u datoteku**.
* **Nakon** što je datoteka **generisana**, **zatvorite** otvoreni prozor **`procmon`** i **otvorite datoteku događaja**.
* Dodajte ove **filtere** i pronaći ćete sve DLL-ove koje je neki **proces pokušao da učita** iz foldera sa upisivim putem sistema:

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### Propušteni DLL-ovi

Pokretanjem ovoga na besplatnoj **virtuelnoj (vmware) Windows 11 mašini** dobio sam ove rezultate:

<figure><img src="../../../.gitbook/assets/image (253).png" alt=""><figcaption></figcaption></figure>

U ovom slučaju, .exe su beskorisni, pa ih zanemarite, propušteni DLL-ovi su bili od:

| Servis                          | DLL                | CMD linija                                                           |
| ------------------------------- | ------------------ | -------------------------------------------------------------------- |
| Task Scheduler (Schedule)       | WptsExtensions.dll | `C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`          |
| Diagnostic Policy Service (DPS) | Unknown.DLL        | `C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork -p -s DPS` |
| ???                             | SharedRes.dll      | `C:\Windows\system32\svchost.exe -k UnistackSvcGroup`                |

Nakon pronalaska ovoga, pronašao sam ovaj zanimljiv blog post koji takođe objašnjava kako [**zloupotrebiti WptsExtensions.dll za eskalaciju privilegija**](https://juggernaut-sec.com/dll-hijacking/#Windows\_10\_Phantom\_DLL\_Hijacking\_-\_WptsExtensionsdll). To je ono što **ćemo sada uraditi**.

### Eksploatacija

Dakle, da bismo **povećali privilegije**, preuzećemo kontrolu nad bibliotekom **WptsExtensions.dll**. Imajući **putanju** i **ime**, samo trebamo **generisati zlonamernu DLL**.

Možete [**pokušati koristiti neki od ovih primera**](./#creating-and-compiling-dlls). Možete pokrenuti payload-ove kao što su: dobijanje reverznog šela, dodavanje korisnika, izvršavanje bekon...

{% hint style="warning" %}
Imajte na umu da **nije svaki servis pokrenut** sa **`NT AUTHORITY\SYSTEM`**, neki se pokreću i sa **`NT AUTHORITY\LOCAL SERVICE`**, koji ima **manje privilegija**, i nećete moći da kreirate novog korisnika i zloupotrebite njegove dozvole.\
Međutim, taj korisnik ima privilegiju **`seImpersonate`**, pa možete koristiti [**potato suite za eskalaciju privilegija**](../roguepotato-and-printspoofer.md). Dakle, u ovom slučaju reverzni šel je bolja opcija od pokušaja kreiranja korisnika.
{% endhint %}

Trenutno, servis **Task Scheduler** se pokreće sa **Nt AUTHORITY\SYSTEM**.

Nakon što ste **generisali zlonamernu DLL** (_u mom slučaju sam koristio x64 reverzni šel i dobio sam povratni šel, ali ga je Defender ubio jer je bio od msfvenom_), sačuvajte je u folderu sa upisivim putem sistema pod imenom **WptsExtensions.dll** i **ponovo pokrenite** računar (ili ponovo pokrenite servis ili uradite šta god je potrebno da se ponovo pokrene pogođeni servis/program).

Kada se servis ponovo pokrene, **dll bi trebalo da se učita i izvrši** (možete **ponovo koristiti** trik sa **procmon**-om da proverite da li je **biblioteka učitana kako je očekivano**).

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
