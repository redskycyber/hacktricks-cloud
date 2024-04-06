# Logstash

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### Logstash

Logstash se koristi za **sakupljanje, transformisanje i slanje logova** kroz sistem poznat kao **pipelines**. Ovi pipelines se sastoje od **input**, **filter** i **output** faza. Interesantan aspekt se javlja kada Logstash radi na kompromitovanoj mašini.

#### Konfiguracija Pipelines-a

Pipelines se konfigurišu u fajlu **/etc/logstash/pipelines.yml**, koji navodi lokacije konfiguracija pipelines-a:

```yaml
# Define your pipelines here. Multiple pipelines can be defined.
# For details on multiple pipelines, refer to the documentation:
# https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: main
path.config: "/etc/logstash/conf.d/*.conf"
- pipeline.id: example
path.config: "/usr/share/logstash/pipeline/1*.conf"
pipeline.workers: 6
```

Ovaj fajl otkriva gde se nalaze **.conf** fajlovi koji sadrže konfiguracije cevovoda. Kada se koristi **Elasticsearch output modul**, često je uobičajeno da **cevovodi** uključuju **Elasticsearch akreditive**, koji često imaju proširene privilegije zbog potrebe Logstash-a da piše podatke u Elasticsearch. Džokere u putanjama konfiguracije omogućavaju Logstash-u da izvrši sve odgovarajuće cevovode u određenom direktorijumu.

#### Eskalacija privilegija putem upisivih cevovoda

Da biste pokušali eskalaciju privilegija, prvo identifikujte korisnika pod kojim se izvršava Logstash servis, obično korisnika **logstash**. Proverite da ispunjavate **jedan** od ovih kriterijuma:

* Imate **pristup za pisanje** fajlu **.conf** cevovoda **ili**
* Fajl **/etc/logstash/pipelines.yml** koristi džokere, i možete pisati u ciljni folder

Dodatno, mora biti ispunjen **jedan** od sledećih uslova:

* Mogućnost restartovanja Logstash servisa **ili**
* Fajl **/etc/logstash/logstash.yml** ima postavljenu opciju **config.reload.automatic: true**

Uzimajući u obzir džokere u konfiguraciji, kreiranje fajla koji odgovara ovom džokeru omogućava izvršavanje komandi. Na primer:

```bash
input {
exec {
command => "whoami"
interval => 120
}
}

output {
file {
path => "/tmp/output.log"
codec => rubydebug
}
}
```

Ovde, **interval** određuje učestalost izvršavanja u sekundama. U datom primeru, komanda **whoami** se izvršava svakih 120 sekundi, a njen izlaz se usmerava u **/tmp/output.log**.

Sa **config.reload.automatic: true** u **/etc/logstash/logstash.yml**, Logstash će automatski detektovati i primeniti nove ili izmenjene konfiguracije cevovoda bez potrebe za ponovnim pokretanjem. Ako nema džokera, i dalje je moguće izmeniti postojeće konfiguracije, ali se savetuje oprez kako bi se izbegle prekide.

### Reference

* [https://insinuator.net/2021/01/pentesting-the-elk-stack/](https://insinuator.net/2021/01/pentesting-the-elk-stack/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **oglašavanje vaše kompanije u HackTricks-u** ili **preuzmete HackTricks u PDF formatu**, proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
