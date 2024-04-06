# macOS .Net Applications Injection

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

**Ovo je sažetak posta** [**https://blog.xpnsec.com/macos-injection-via-third-party-frameworks/**](https://blog.xpnsec.com/macos-injection-via-third-party-frameworks/)**. Pogledajte ga za dalje detalje!**

## .NET Core Debugiranje <a href="#net-core-debugging" id="net-core-debugging"></a>

### **Uspostavljanje debugiranja** <a href="#net-core-debugging" id="net-core-debugging"></a>

Komunikacija između debugera i debugiranog programa u .NET-u se upravlja preko [**dbgtransportsession.cpp**](https://github.com/dotnet/runtime/blob/0633ecfb79a3b2f1e4c098d1dd0166bc1ae41739/src/coreclr/debug/shared/dbgtransportsession.cpp). Ovaj komponenta postavlja dve nazvane cevi po .NET procesu, kako je prikazano u [dbgtransportsession.cpp#L127](https://github.com/dotnet/runtime/blob/0633ecfb79a3b2f1e4c098d1dd0166bc1ae41739/src/coreclr/debug/shared/dbgtransportsession.cpp#L127), koje se iniciraju preko [twowaypipe.cpp#L27](https://github.com/dotnet/runtime/blob/0633ecfb79a3b2f1e4c098d1dd0166bc1ae41739/src/coreclr/debug/debug-pal/unix/twowaypipe.cpp#L27). Ove cevi su sufiksirane sa **`-in`** i **`-out`**.

Posetom korisnikovom **`$TMPDIR`**, mogu se pronaći FIFO-ovi za debugiranje .Net aplikacija.

[**DbgTransportSession::TransportWorker**](https://github.com/dotnet/runtime/blob/0633ecfb79a3b2f1e4c098d1dd0166bc1ae41739/src/coreclr/debug/shared/dbgtransportsession.cpp#L1259) je odgovoran za upravljanje komunikacijom od debugera. Da bi započeo novu sesiju debugiranja, debugger mora poslati poruku putem `out` cevi koja počinje sa `MessageHeader` strukturom, detaljno opisanom u izvornom kodu .NET-a:

```c
struct MessageHeader {
MessageType   m_eType;        // Message type
DWORD         m_cbDataBlock;  // Size of following data block (can be zero)
DWORD         m_dwId;         // Message ID from sender
DWORD         m_dwReplyId;    // Reply-to Message ID
DWORD         m_dwLastSeenId; // Last seen Message ID by sender
DWORD         m_dwReserved;   // Reserved for future (initialize to zero)
union {
struct {
DWORD         m_dwMajorVersion;   // Requested/accepted protocol version
DWORD         m_dwMinorVersion;
} VersionInfo;
...
} TypeSpecificData;
BYTE          m_sMustBeZero[8];
}
```

Da biste zatražili novu sesiju, ova struktura se popunjava na sledeći način, postavljajući tip poruke na `MT_SessionRequest` i verziju protokola na trenutnu verziju:

```c
static const DWORD kCurrentMajorVersion = 2;
static const DWORD kCurrentMinorVersion = 0;

// Configure the message type and version
sSendHeader.m_eType = MT_SessionRequest;
sSendHeader.TypeSpecificData.VersionInfo.m_dwMajorVersion = kCurrentMajorVersion;
sSendHeader.TypeSpecificData.VersionInfo.m_dwMinorVersion = kCurrentMinorVersion;
sSendHeader.m_cbDataBlock = sizeof(SessionRequestData);
```

Ova zaglavlje se zatim šalje cilju koristeći `write` sistemski poziv, praćeno strukturom `sessionRequestData` koja sadrži GUID za sesiju:

```c
write(wr, &sSendHeader, sizeof(MessageHeader));
memset(&sDataBlock.m_sSessionID, 9, sizeof(SessionRequestData));
write(wr, &sDataBlock, sizeof(SessionRequestData));
```

Operacija čitanja na `out` cevi potvrđuje uspeh ili neuspeh uspostavljanja sesije za debagovanje:

```c
read(rd, &sReceiveHeader, sizeof(MessageHeader));
```

## Čitanje memorije

Jednom kada je uspostavljena sesija za debagovanje, memorija se može čitati koristeći tip poruke [`MT_ReadMemory`](https://github.com/dotnet/runtime/blob/f3a45a91441cf938765bafc795cbf4885cad8800/src/coreclr/src/debug/shared/dbgtransportsession.cpp#L1896). Funkcija readMemory je detaljno opisana, izvodeći neophodne korake za slanje zahteva za čitanje i dobijanje odgovora:

```c
bool readMemory(void *addr, int len, unsigned char **output) {
// Allocation and initialization
...
// Write header and read response
...
// Read the memory from the debuggee
...
return true;
}
```

Potpuni dokaz koncepta (POC) dostupan je [ovde](https://gist.github.com/xpn/95eefc14918998853f6e0ab48d9f7b0b).

## Pisanje u memoriju

Slično tome, memorija se može pisati pomoću funkcije `writeMemory`. Postupak uključuje postavljanje tipa poruke na `MT_WriteMemory`, navođenje adrese i dužine podataka, a zatim slanje podataka:

```c
bool writeMemory(void *addr, int len, unsigned char *input) {
// Increment IDs, set message type, and specify memory location
...
// Write header and data, then read the response
...
// Confirm memory write was successful
...
return true;
}
```

Povezani POC je dostupan [ovde](https://gist.github.com/xpn/7c3040a7398808747e158a25745380a5).

## Izvršavanje koda u .NET Core <a href="#net-core-code-execution" id="net-core-code-execution"></a>

Da biste izvršili kod, potrebno je identifikovati memorijsku regiju sa dozvolama za čitanje, pisanje i izvršavanje (rwx), što se može uraditi pomoću komande vmmap -pages:

```bash
vmmap -pages [pid]
vmmap -pages 35829 | grep "rwx/rwx"
```

Lociranje mesta za prepisivanje pokazivača funkcije je neophodno, a u .NET Core-u to se može postići ciljanjem **Dynamic Function Table (DFT)**. Ova tabela, detaljno opisana u [`jithelpers.h`](https://github.com/dotnet/runtime/blob/6072e4d3a7a2a1493f514cdf4be75a3d56580e84/src/coreclr/src/inc/jithelpers.h), koristi se od strane izvršnog okruženja za JIT kompilacijske pomoćne funkcije.

Za x64 sisteme, može se koristiti pretraga potpisa kako bi se pronašla referenca na simbol `_hlpDynamicFuncTable` u `libcorclr.dll`.

Debugger funkcija `MT_GetDCB` pruža korisne informacije, uključujući adresu pomoćne funkcije `m_helperRemoteStartAddr`, koja ukazuje na lokaciju `libcorclr.dll` u memoriji procesa. Ova adresa se zatim koristi za pretragu DFT i prepisivanje pokazivača funkcije sa adresom shell koda.

Ceo POC kod za ubacivanje u PowerShell može se pronaći [ovde](https://gist.github.com/xpn/b427998c8b3924ab1d63c89d273734b6).

## Reference

* [https://blog.xpnsec.com/macos-injection-via-third-party-frameworks/](https://blog.xpnsec.com/macos-injection-via-third-party-frameworks/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
