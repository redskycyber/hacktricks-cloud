# macOS Kernel Extensions

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za sajber bezbednost**? Želite da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite pristup **poslednjoj verziji PEASS-a ili preuzimanje HackTricks-a u PDF formatu**? Pogledajte [**PLANOVE ZA ČLANSTVO**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu ekskluzivnu kolekciju [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite **zvanični PEASS i HackTricks** [**swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) **Discord grupi** ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Podelite svoje hakovanje trikove slanjem PR-a na** [**hacktricks repozitorijum**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijum**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Osnovne informacije

Kernel ekstenzije (Kexts) su **paketi** sa ekstenzijom **`.kext`** koji se **direktno učitavaju u macOS kernel prostor**, pružajući dodatne funkcionalnosti glavnom operativnom sistemu.

### Zahtevi

Očigledno, ovo je toliko moćno da je **komplikovano učitati kernel ekstenziju**. Ovo su **zahtevi** koje kernel ekstenzija mora ispuniti da bi bila učitana:

* Prilikom **ulaska u režim oporavka**, kernel **ekstenzije moraju biti dozvoljene** za učitavanje:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Kernel ekstenzija mora biti **potpisana kernel potpisnim sertifikatom**, koji može **dodeliti samo Apple**. Ko će detaljno pregledati kompaniju i razloge zašto je potrebno.
* Kernel ekstenzija takođe mora biti **notarizovana**, Apple će moći da je proveri na prisustvo malvera.
* Zatim, **root** korisnik je taj koji može **učitati kernel ekstenziju** i fajlovi unutar paketa moraju **pripadati root-u**.
* Tokom procesa učitavanja, paket mora biti pripremljen na **zaštićenoj lokaciji koja nije root**: `/Library/StagedExtensions` (zahteva `com.apple.rootless.storage.KernelExtensionManagement` dozvolu).
* Na kraju, prilikom pokušaja učitavanja, korisnik će [**dobiti zahtev za potvrdu**](https://developer.apple.com/library/archive/technotes/tn2459/\_index.html) i, ako se prihvati, računar mora biti **restartovan** da bi se učitao.

### Proces učitavanja

U Catalina verziji je bilo ovako: Zanimljivo je napomenuti da se **proces verifikacije** dešava u **userland-u**. Međutim, samo aplikacije sa **`com.apple.private.security.kext-management`** dozvolom mogu **zatražiti od kernela da učita ekstenziju**: `kextcache`, `kextload`, `kextutil`, `kextd`, `syspolicyd`

1. **`kextutil`** cli **pokreće** proces **verifikacije** za učitavanje ekstenzije

* Razgovaraće sa **`kextd`** slanjem putem **Mach servisa**.

2. **`kextd`** će proveriti nekoliko stvari, kao što je **potpis**

* Razgovaraće sa **`syspolicyd`**-om da **proveri** da li se ekstenzija može **učitati**.

3. **`syspolicyd`** će **zatražiti od korisnika** potvrdu ako ekstenzija prethodno nije učitana.

* **`syspolicyd`** će prijaviti rezultat **`kextd`**-u

4. **`kextd`** će na kraju moći da **kaže kernelu da učita** ekstenziju

Ako **`kextd`** nije dostupan, **`kextutil`** može obaviti iste provere.

## References

* [https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/](https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/)
* [https://www.youtube.com/watch?v=hGKOskSiaQo](https://www.youtube.com/watch?v=hGKOskSiaQo)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za sajber bezbednost**? Želite da vidite svoju **kompaniju reklamiranu na HackTricks-u**? Ili želite pristup **poslednjoj verziji PEASS-a ili preuzimanje HackTricks-a u PDF formatu**? Pogledajte [**PLANOVE ZA ČLANSTVO**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu ekskluzivnu kolekciju [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite **zvanični PEASS i HackTricks** [**swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) **Discord grupi** ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Podelite svoje hakovanje trikove slanjem PR-a na** [**hacktricks repozitorijum**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijum**](https://github.com/carlospolop/hacktricks-cloud).

</details>
