# macOS Memory Dumping

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Memorija Artifakti

### Swap Fajlovi

Swap fajlovi, kao što je `/private/var/vm/swapfile0`, služe kao **keš kada je fizička memorija puna**. Kada nema više mesta u fizičkoj memoriji, njeni podaci se prenose u swap fajl i zatim se po potrebi vraćaju u fizičku memoriju. Može biti prisutno više swap fajlova, sa imenima kao što su swapfile0, swapfile1, i tako dalje.

### Hibernate Slika

Fajl koji se nalazi na lokaciji `/private/var/vm/sleepimage` je ključan tokom **hibernacije**. **Podaci iz memorije se čuvaju u ovom fajlu kada OS X hibernira**. Nakon buđenja računara, sistem iz ovog fajla povlači podatke iz memorije, omogućavajući korisniku da nastavi gde je stao.

Važno je napomenuti da je na modernim MacOS sistemima ovaj fajl obično enkriptovan iz bezbednosnih razloga, što otežava oporavak.

* Da biste proverili da li je enkripcija omogućena za sleepimage, može se pokrenuti komanda `sysctl vm.swapusage`. Ovo će pokazati da li je fajl enkriptovan.

### Logovi o pritisku na memoriju

Još jedan važan fajl vezan za memoriju u MacOS sistemima su **logovi o pritisku na memoriju**. Ovi logovi se nalaze u `/var/log` i sadrže detaljne informacije o korišćenju memorije sistema i događajima pritiska na memoriju. Mogu biti posebno korisni za dijagnostikovanje problema vezanih za memoriju ili razumevanje načina na koji sistem upravlja memorijom tokom vremena.

## Dumpiranje memorije pomoću osxpmem

Da biste dumpirali memoriju na MacOS mašini, možete koristiti [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip).

**Napomena**: Sledeće instrukcije će raditi samo za Mac računare sa Intel arhitekturom. Ovaj alat je sada arhiviran i poslednje izdanje je bilo 2017. godine. Binarna datoteka preuzeta koristeći dole navedene instrukcije je namenjena Intel čipovima, jer Apple Silicon nije postojao 2017. godine. Moguće je da je moguće kompajlirati binarnu datoteku za arm64 arhitekturu, ali to ćete morati sami da isprobate.

```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```

Ako pronađete ovu grešku: `osxpmem.app/MacPmem.kext nije uspeo da se učita - (libkern/kext) autentifikacija nije uspela (vlasništvo/datoteke/dozvole); proverite sistemske/kernel dnevnike za greške ili pokušajte sa kextutil(8)` Možete je popraviti na sledeći način:

```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```

**Drugi problemi** mogu biti rešeni **omogućavanjem učitavanja kext-a** u "Security & Privacy --> General", samo ga **omogućite**.

Takođe možete koristiti ovaj **oneliner** za preuzimanje aplikacije, učitavanje kext-a i ispuštanje memorije:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
