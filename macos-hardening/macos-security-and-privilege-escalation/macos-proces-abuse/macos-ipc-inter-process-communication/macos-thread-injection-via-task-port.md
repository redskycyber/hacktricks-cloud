# macOS Thread Injection via Task port

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Kod

* [https://github.com/bazad/threadexec](https://github.com/bazad/threadexec)
* [https://gist.github.com/knightsc/bd6dfeccb02b77eb6409db5601dcef36](https://gist.github.com/knightsc/bd6dfeccb02b77eb6409db5601dcef36)

## 1. Hakovanje niti

Prvo se poziva funkcija **`task_threads()`** na Task portu da bi se dobio spisak niti iz udaljenog taska. Nit se bira za hakovanje. Ovaj pristup se razlikuje od konvencionalnih metoda ubacivanja koda jer je stvaranje nove udaljene niti zabranjeno zbog nove zaštite koja blokira `thread_create_running()`.

Da bi se kontrolisala nit, poziva se funkcija **`thread_suspend()`**, koja zaustavlja njeno izvršavanje.

Jedine dozvoljene operacije na udaljenoj niti uključuju **zaustavljanje** i **pokretanje** niti, **dobijanje** i **menjanje** vrednosti registara. Udaljeni pozivi funkcija se pokreću postavljanjem registara `x0` do `x7` na **argumente**, konfigurisanjem **`pc`** da cilja željenu funkciju i aktiviranjem niti. Da bi se osiguralo da nit ne padne nakon povratka, neophodno je otkriti povratnu vrednost.

Jedna strategija uključuje **registrovanje rukovaoca izuzecima** za udaljenu nit korišćenjem `thread_set_exception_ports()`, postavljanje registra `lr` na nevažeću adresu pre poziva funkcije. Ovo pokreće izuzetak nakon izvršavanja funkcije, šalje poruku na port izuzetka i omogućava inspekciju stanja niti radi povraćaja povratne vrednosti. Alternativno, kao što je preuzeto iz Ian Beer-ovog triple\_fetch exploit-a, `lr` je postavljen da beskonačno petlja. Registri niti se zatim neprekidno prate dok **`pc` ne pokazuje na tu instrukciju**.

## 2. Mach portovi za komunikaciju

Sledeća faza uključuje uspostavljanje Mach portova radi olakšane komunikacije sa udaljenom niti. Ovi portovi su od suštinskog značaja za prenos proizvoljnih prava slanja i primanja između zadataka.

Za dvosmernu komunikaciju, kreiraju se dva Mach primanja prava: jedno u lokalnom, a drugo u udaljenom zadatku. Zatim se za svaki port prenosi pravo slanja na odgovarajući zadatak, omogućavajući razmenu poruka.

Fokus je na lokalnom portu, gde lokalni zadatak drži primanje prava. Port se kreira pomoću `mach_port_allocate()`. Izazov leži u prenosu prava slanja na ovaj port u udaljeni zadatak.

Jedna strategija uključuje iskorišćavanje `thread_set_special_port()` da bi se postavilo pravo slanja na lokalni port u `THREAD_KERNEL_PORT` udaljene niti. Zatim se udaljenoj niti nalaže da pozove `mach_thread_self()` da bi dobila pravo slanja.

Za udaljeni port, proces je suštinski obrnut. Udaljenoj niti se nalaže da generiše Mach port putem `mach_reply_port()` (jer `mach_port_allocate()` nije pogodan zbog mehanizma povratka). Nakon kreiranja porta, u udaljenoj niti se poziva `mach_port_insert_right()` da bi se uspostavilo pravo slanja. Ovo pravo se zatim smešta u kernel pomoću `thread_set_special_port()`. U lokalnom zadatku se zatim koristi `thread_get_special_port()` na udaljenoj niti da bi se dobilo pravo slanja na novo dodeljeni Mach port u udaljenom zadatku.

Završetak ovih koraka rezultira uspostavljanjem Mach portova, postavljajući osnovu za dvosmernu komunikaciju.

## 3. Osnovni primitivi za čitanje/pisanje memorije

U ovoj sekciji fokus je na korišćenju izvršnog primitiva za uspostavljanje osnovnih primitiva za čitanje i pisanje memorije. Ovi početni koraci su ključni za dobijanje veće kontrole nad udaljenim procesom, iako primitivi u ovoj fazi neće služiti mnogo svrsi. Uskoro će biti nadograđeni na naprednije verzije.

### Čitanje i pisanje memorije korišćenjem izvršnog primitiva

Cilj je izvršiti čitanje i pisanje memorije koristeći određene funkcije. Za čitanje memorije koriste se funkcije slične sledećoj strukturi:

```c
uint64_t read_func(uint64_t *address) {
return *address;
}
```

I za pisanje u memoriju, koriste se funkcije slične ovoj strukturi:

```c
void write_func(uint64_t *address, uint64_t value) {
*address = value;
}
```

Ove funkcije odgovaraju datim sklopovskim instrukcijama:

```
_read_func:
ldr x0, [x0]
ret
_write_func:
str x1, [x0]
ret
```

### Identifikacija odgovarajućih funkcija

Skeniranje uobičajenih biblioteka otkrilo je odgovarajuće kandidate za ove operacije:

1. **Čitanje memorije:** Funkcija `property_getName()` iz [Objective-C runtime biblioteke](https://opensource.apple.com/source/objc4/objc4-723/runtime/objc-runtime-new.mm.auto.html) identifikovana je kao odgovarajuća funkcija za čitanje memorije. Funkcija je prikazana ispod:

```c
const char *property_getName(objc_property_t prop) {
return prop->name;
}
```

Ova funkcija efektivno deluje kao `read_func` vraćajući prvo polje `objc_property_t`.

2. **Pisanje u memoriju:** Pronalaženje prethodno izgrađene funkcije za pisanje u memoriju je izazovnije. Međutim, funkcija `_xpc_int64_set_value()` iz libxpc je odgovarajući kandidat sa sledećim rastavljanjem:

```c
__xpc_int64_set_value:
str x1, [x0, #0x18]
ret
```

Da biste izvršili 64-bitni upis na određenoj adresi, udaljeni poziv je strukturiran na sledeći način:

```c
_xpc_int64_set_value(address - 0x18, value)
```

Sa ovim osnovama postavljenim, stvorena je osnova za stvaranje deljene memorije, što predstavlja značajan napredak u kontroli udaljenog procesa.

## 4. Postavljanje deljene memorije

Cilj je uspostaviti deljenu memoriju između lokalnih i udaljenih zadataka, olakšavajući prenos podataka i omogućavajući pozivanje funkcija sa više argumenata. Pristup uključuje korišćenje `libxpc` i njenog objekta tipa `OS_xpc_shmem`, koji se zasniva na unosima memorije Mach.

### Pregled procesa:

1. **Alokacija memorije**:

* Alokacija memorije za deljenje korišćenjem `mach_vm_allocate()`.
* Korišćenje `xpc_shmem_create()` za kreiranje objekta `OS_xpc_shmem` za alocirani region memorije. Ova funkcija će upravljati kreiranjem unosa memorije Mach i čuvati Mach send pravo na offsetu `0x18` objekta `OS_xpc_shmem`.

2. **Kreiranje deljene memorije u udaljenom procesu**:

* Alokacija memorije za objekat `OS_xpc_shmem` u udaljenom procesu pomoću udaljenog poziva `malloc()`.
* Kopiranje sadržaja lokalnog objekta `OS_xpc_shmem` u udaljeni proces. Međutim, ova početna kopija će imati netačna imena unosa memorije Mach na offsetu `0x18`.

3. **Ispravljanje unosa memorije Mach**:

* Koristite metodu `thread_set_special_port()` za umetanje send prava za unos memorije Mach u udaljeni zadatak.
* Ispravite polje unosa memorije Mach na offsetu `0x18` tako što ćete ga prepisati imenom unosa memorije udaljenog zadatka.

4. **Završno postavljanje deljene memorije**:

* Validirajte udaljeni objekat `OS_xpc_shmem`.
* Ustanovite mapiranje deljene memorije pomoću udaljenog poziva `xpc_shmem_remote()`.

Prateći ove korake, deljena memorija između lokalnih i udaljenih zadataka će biti efikasno postavljena, omogućavajući jednostavan prenos podataka i izvršavanje funkcija koje zahtevaju više argumenata.

## Dodatni isečci koda

Za alokaciju memorije i kreiranje objekta deljene memorije:

```c
mach_vm_allocate();
xpc_shmem_create();
```

Za kreiranje i ispravljanje objekta deljene memorije u udaljenom procesu:

```c
malloc(); // for allocating memory remotely
thread_set_special_port(); // for inserting send right
```

Zapamtite da pravilno rukujete detaljima Mach portova i imenima memorijskih unosa kako biste osigurali ispravno funkcionisanje podele deljene memorije.

## 5. Postizanje potpune kontrole

Nakon uspešnog uspostavljanja deljene memorije i sticanja proizvoljnih izvršnih mogućnosti, suštinski smo stekli potpunu kontrolu nad ciljnim procesom. Ključne funkcionalnosti koje omogućavaju ovu kontrolu su:

1. **Proizvoljne operacije nad memorijom**:

* Izvršite proizvoljno čitanje memorije pozivanjem `memcpy()` funkcije za kopiranje podataka iz deljenog regiona.
* Izvršite proizvoljno pisanje memorije koristeći `memcpy()` funkciju za prenos podataka u deljeni region.

2. **Rukovanje pozivima funkcija sa više argumenata**:

* Za funkcije koje zahtevaju više od 8 argumenata, rasporedite dodatne argumente na steku u skladu sa konvencijom pozivanja.

3. **Prenos Mach portova**:

* Prenosite Mach portove između zadataka putem Mach poruka putem prethodno uspostavljenih portova.

4. **Prenos deskriptora fajlova**:

* Prenosite deskriptore fajlova između procesa koristeći fileportove, tehniku istaknutu od strane Iana Beera u `triple_fetch`.

Ova sveobuhvatna kontrola je obuhvaćena bibliotekom [threadexec](https://github.com/bazad/threadexec), koja pruža detaljnu implementaciju i korisnički prijateljski API za interakciju sa ciljnim procesom.

## Važne razmatranja:

* Obezbedite pravilnu upotrebu `memcpy()` funkcije za operacije čitanja/pisanja memorije kako biste održali stabilnost sistema i integritet podataka.
* Prilikom prenosa Mach portova ili deskriptora fajlova, pratite odgovarajuće protokole i odgovorno rukujte resursima kako biste sprečili curenje ili neželjeni pristup.

Prateći ove smernice i koristeći biblioteku `threadexec`, možete efikasno upravljati i interagovati sa procesima na detaljnom nivou, postižući potpunu kontrolu nad ciljnim procesom.

## Reference

* [https://bazad.github.io/2018/10/bypassing-platform-binary-task-threads/](https://bazad.github.io/2018/10/bypassing-platform-binary-task-threads/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Pogledajte [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
