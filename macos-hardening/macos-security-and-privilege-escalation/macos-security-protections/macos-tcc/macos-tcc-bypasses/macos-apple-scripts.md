# macOS Apple Scripts

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Apple skripte

To je skriptni jezik koji se koristi za automatizaciju zadataka **interakcijom sa udaljenim procesima**. Olakšava **zahtevanje drugih procesa da izvrše određene radnje**. **Malver** može zloupotrebiti ove funkcije kako bi zloupotrebio funkcije izvožene od strane drugih procesa.\
Na primer, malver bi mogao **ubaciti proizvoljni JS kod na otvorene stranice pregledača**. Ili **automatski kliknuti** na dozvole koje korisniku traže.

```applescript
tell window 1 of process "SecurityAgent"
click button "Always Allow" of group 1
end tell
```

Evo nekoliko primera: [https://github.com/abbeycode/AppleScripts](https://github.com/abbeycode/AppleScripts)\
Pronađite više informacija o malveru koji koristi Apple skripte [**ovde**](https://www.sentinelone.com/blog/how-offensive-actors-use-applescript-for-attacking-macos/).

Apple skripte se mogu lako "**kompajlirati**". Ove verzije se mogu lako "**dekompajlirati**" pomoću `osadecompile`.

Međutim, ove skripte takođe mogu biti **izvežene kao "Samo za čitanje"** (putem opcije "Izvezi..."):

<figure><img src="https://github.com/carlospolop/hacktricks/raw/master/.gitbook/assets/image%20(556).png" alt=""><figcaption></figcaption></figure>

\`\`\` file mal.scpt mal.scpt: AppleScript compiled \`\`\` U ovom slučaju sadržaj ne može biti dekompiliran čak ni sa \`osadecompile\`.

Međutim, i dalje postoje alati koji se mogu koristiti za razumevanje ovakvih izvršnih datoteka, [**pročitajte ovaj istraživački rad za više informacija**](https://labs.sentinelone.com/fade-dead-adventures-in-reversing-malicious-run-only-applescripts/)). Alat [**applescript-disassembler**](https://github.com/Jinmo/applescript-disassembler) sa [**aevt\_decompile**](https://github.com/SentineLabs/aevt\_decompile) će biti veoma koristan za razumevanje kako skripta radi.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini da podržite HackTricks:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
