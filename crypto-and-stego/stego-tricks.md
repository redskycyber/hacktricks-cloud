# Stego Tricks

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJE**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## **Izdvajanje Podataka iz Fajlova**

### **Binwalk**

Alat za pretragu binarnih fajlova za ugrađene skrivene fajlove i podatke. Instalira se putem `apt` i njegov izvorni kod je dostupan na [GitHub-u](https://github.com/ReFirmLabs/binwalk).

```bash
binwalk file # Displays the embedded data
binwalk -e file # Extracts the data
binwalk --dd ".*" file # Extracts all data
```

### **Foremost**

Vraća datoteke na osnovu njihovih zaglavlja i podnožja, korisno za png slike. Instalira se putem `apt` sa izvorom na [GitHub-u](https://github.com/korczis/foremost).

```bash
foremost -i file # Extracts data
```

### **Exiftool**

Pomaže u pregledu metapodataka datoteke, dostupan [ovde](https://www.sno.phy.queensu.ca/\~phil/exiftool/).

```bash
exiftool file # Shows the metadata
```

### **Exiv2**

Slično kao exiftool, za pregled metapodataka. Instalira se putem `apt`, izvorni kod na [GitHub-u](https://github.com/Exiv2/exiv2), i ima [zvaničnu veb stranicu](http://www.exiv2.org/).

```bash
exiv2 file # Shows the metadata
```

### **Fajl**

Identifikujte tip fajla sa kojim radite.

### **Niske**

Izdvaja čitljive niske iz fajlova, koristeći različite postavke enkodiranja za filtriranje rezultata.

```bash
strings -n 6 file # Extracts strings with a minimum length of 6
strings -n 6 file | head -n 20 # First 20 strings
strings -n 6 file | tail -n 20 # Last 20 strings
strings -e s -n 6 file # 7bit strings
strings -e S -n 6 file # 8bit strings
strings -e l -n 6 file # 16bit strings (little-endian)
strings -e b -n 6 file # 16bit strings (big-endian)
strings -e L -n 6 file # 32bit strings (little-endian)
strings -e B -n 6 file # 32bit strings (big-endian)
```

### **Poređenje (cmp)**

Korisno za upoređivanje modifikovanog fajla sa originalnom verzijom pronađenom online.

```bash
cmp original.jpg stego.jpg -b -l
```

## **Izdvajanje skrivenih podataka u tekstu**

### **Skriveni podaci u razmacima**

Nevidljivi znakovi u navodno praznim prostorima mogu sakriti informacije. Da biste izvukli ove podatke, posetite [https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder](https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder).

## **Izdvajanje podataka iz slika**

### **Identifikacija detalja slike pomoću GraphicMagick-a**

[GraphicMagick](https://imagemagick.org/script/download.php) služi za određivanje vrsta datoteka slika i identifikaciju potencijalne korupcije. Izvršite naredbu ispod da biste pregledali sliku:

```bash
./magick identify -verbose stego.jpg
```

Da biste pokušali popraviti oštećenu sliku, dodavanje metapodataka u komentaru može pomoći:

```bash
./magick mogrify -set comment 'Extraneous bytes removed' stego.jpg
```

### **Steghide za skrivanje podataka**

Steghide olakšava skrivanje podataka unutar `JPEG, BMP, WAV i AU` datoteka, sposoban je za ugradnju i izvlačenje šifrovanih podataka. Instalacija je jednostavna korišćenjem `apt`, a njen [izvorni kod je dostupan na GitHub-u](https://github.com/StefanoDeVuono/steghide).

**Komande:**

* `steghide info file` otkriva da li datoteka sadrži skrivene podatke.
* `steghide extract -sf file [--passphrase password]` izvlači skrivene podatke, lozinka je opcionalna.

Za web bazirano izvlačenje, posetite [ovaj veb sajt](https://futureboy.us/stegano/decinput.html).

**Bruteforce napad sa Stegcracker-om:**

* Za pokušaj probijanja lozinke na Steghide-u, koristite [stegcracker](https://github.com/Paradoxis/StegCracker.git) na sledeći način:

```bash
stegcracker <file> [<wordlist>]
```

### **zsteg za PNG i BMP datoteke**

zsteg se specijalizuje za otkrivanje skrivenih podataka u PNG i BMP datotekama. Instalacija se vrši putem `gem install zsteg`, a izvor možete pronaći na [GitHub-u](https://github.com/zed-0xff/zsteg).

**Komande:**

* `zsteg -a file` primenjuje sve metode detekcije na datoteku.
* `zsteg -E file` specificira payload za ekstrakciju podataka.

### **StegoVeritas i Stegsolve**

**stegoVeritas** proverava metapodatke, vrši transformacije slike i primenjuje LSB brute force među ostalim funkcijama. Koristite `stegoveritas.py -h` za punu listu opcija i `stegoveritas.py stego.jpg` za izvršavanje svih provera.

**Stegsolve** primenjuje različite filtere boja kako bi otkrio skrivene tekstove ili poruke unutar slika. Dostupan je na [GitHub-u](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve/stegsolve).

### **FFT za Otkrivanje Skrivenog Sadržaja**

Tehnike brze Furijeove transformacije (FFT) mogu otkriti skriveni sadržaj u slikama. Korisni resursi uključuju:

* [EPFL Demo](http://bigwww.epfl.ch/demo/ip/demos/FFT/)
* [Ejectamenta](https://www.ejectamenta.com/Fourifier-fullscreen/)
* [FFTStegPic na GitHub-u](https://github.com/0xcomposure/FFTStegPic)

### **Stegpy za Audio i Image Datoteke**

Stegpy omogućava ugradnju informacija u slikovne i audio datoteke, podržavajući formate poput PNG, BMP, GIF, WebP i WAV. Dostupan je na [GitHub-u](https://github.com/dhsdshdhk/stegpy).

### **Pngcheck za Analizu PNG Datoteka**

Za analizu PNG datoteka ili proveru njihove autentičnosti, koristite:

```bash
apt-get install pngcheck
pngcheck stego.png
```

### **Dodatni alati za analizu slika**

Za dalje istraživanje, razmotrite posetu:

* [Magic Eye Solver](http://magiceye.ecksdee.co.uk/)
* [Analiza nivoa greške slike](https://29a.ch/sandbox/2012/imageerrorlevelanalysis/)
* [Outguess](https://github.com/resurrecting-open-source-projects/outguess)
* [OpenStego](https://www.openstego.com/)
* [DIIT](https://diit.sourceforge.net/)

## **Izdvajanje podataka iz audio zapisa**

**Audio steganografija** nudi jedinstvenu metodu za skrivanje informacija unutar zvučnih datoteka. Različiti alati se koriste za ugradnju ili dobijanje skrivenog sadržaja.

### **Steghide (JPEG, BMP, WAV, AU)**

Steghide je svestran alat dizajniran za skrivanje podataka u JPEG, BMP, WAV i AU datotekama. Detaljne instrukcije su dostupne u [dokumentaciji trikova steganografije](stego-tricks.md#steghide).

### **Stegpy (PNG, BMP, GIF, WebP, WAV)**

Ovaj alat je kompatibilan sa različitim formatima uključujući PNG, BMP, GIF, WebP i WAV. Za više informacija, pogledajte [odeljak o Stegpy-u](stego-tricks.md#stegpy-png-bmp-gif-webp-wav).

### **ffmpeg**

ffmpeg je ključan za procenu integriteta audio datoteka, ističući detaljne informacije i otkrivajući eventualne neslaganja.

```bash
ffmpeg -v info -i stego.mp3 -f null -
```

### **WavSteg (WAV)**

WavSteg se ističe u skrivanju i izvlačenju podataka unutar WAV fajlova koristeći strategiju najmanje značajnog bita. Dostupan je na [GitHub](https://github.com/ragibson/Steganography#WavSteg). Komande uključuju:

```bash
python3 WavSteg.py -r -b 1 -s soundfile -o outputfile

python3 WavSteg.py -r -b 2 -s soundfile -o outputfile
```

### **Deepsound**

Deepsound omogućava šifrovanje i otkrivanje informacija unutar zvučnih fajlova koristeći AES-256. Može se preuzeti sa [zvanične stranice](http://jpinsoft.net/deepsound/download.aspx).

### **Sonic Visualizer**

Neprocenjiv alat za vizuelnu i analitičku inspekciju audio fajlova, Sonic Visualizer može otkriti skrivene elemente koji nisu detektovani na druge načine. Posetite [zvanični sajt](https://www.sonicvisualiser.org/) za više informacija.

### **DTMF Tones - Dial Tones**

Detekcija DTMF tonova u audio fajlovima može se postići korišćenjem online alata poput [ovog DTMF detektora](https://unframework.github.io/dtmf-detect/) i [DialABC](http://dialabc.com/sound/detect/index.html).

## **Other Techniques**

### **Binary Length SQRT - QR Code**

Binarni podaci koji se kvadriraju u ceo broj mogu predstavljati QR kod. Koristite ovaj isječak koda za proveru:

```python
import math
math.sqrt(2500) #50
```

### **Prevod na srpski**

Za konverziju binarnog u sliku, proverite [dcode](https://www.dcode.fr/binary-image). Za čitanje QR kodova, koristite [ovaj online čitač barkoda](https://online-barcode-reader.inliteresearch.com/).

### **Prevod Brailove azbuke**

Za prevod Brailove azbuke, [Branah Braille Translator](https://www.branah.com/braille-translator) je odličan resurs.

## **Reference**

* [**https://0xrick.github.io/lists/stego/**](https://0xrick.github.io/lists/stego/)
* [**https://github.com/DominicBreuker/stego-toolkit**](https://github.com/DominicBreuker/stego-toolkit)

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od početnika do stručnjaka sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
