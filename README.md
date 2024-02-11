# HackTricks Cloud

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLAN SUBSKRYPCJI**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** mnie na **Twitterze** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów GitHub.

</details>

<figure><img src=".gitbook/assets/cloud.gif" alt=""><figcaption></figcaption></figure>

_Loga i animacje Hacktricks zaprojektowane przez_ [_@ppiernacho_](https://www.instagram.com/ppieranacho/)_._

{% hint style="success" %}
Witaj na stronie, na której znajdziesz każdą **sztuczkę/hakowanie/technikę związaną z CI/CD i chmurą**, którą nauczyłem się w **CTF-ach**, **rzeczywistych** środowiskach **produkcyjnych**, **badaniach** i **czytaniu** badań i wiadomości.
{% endhint %}

## **Metodologia testowania penetracyjnego CI/CD**

**W metodologii HackTricks CI/CD znajdziesz, jak testować penetracyjnie infrastrukturę związana z działaniami CI/CD.** Przeczytaj następującą stronę dla **wprowadzenia:**

{% content-ref url="pentesting-ci-cd/pentesting-ci-cd-methodology.md" %}
[pentesting-ci-cd-methodology.md](pentesting-ci-cd/pentesting-ci-cd-methodology.md)
{% endcontent-ref %}

## Metodologia testowania penetracyjnego chmury

**W metodologii HackTricks Cloud znajdziesz, jak testować penetracyjnie środowiska chmurowe.** Przeczytaj następującą stronę dla **wprowadzenia:**

{% content-ref url="pentesting-cloud/pentesting-cloud-methodology.md" %}
[pentesting-cloud-methodology.md](pentesting-cloud/pentesting-cloud-methodology.md)
{% endcontent-ref %}

## Licencja

**Prawa autorskie © Carlos Polop 2023. O ile nie określono inaczej (informacje zewnętrzne skopiowane do książki należą do pierwotnych autorów), tekst na stronie** [**HACK TRICKS CLOUD**](https://github.com/carlospolop/hacktricks-cloud) **autorstwa Carlosa Polopa jest licencjonowany na zasadach**[ **Uznanie autorstwa-Użycie niekomercyjne 4.0 Międzynarodowe (CC BY-NC 4.0)**](https://creativecommons.org/licenses/by-nc/4.0/)**.**\
**Jeśli chcesz go używać w celach komercyjnych, skontaktuj się ze mną.**

## **Oświadczenie**

{% hint style="danger" %}
Ta książka, 'HackTricks Cloud', ma charakter edukacyjny i informacyjny. Zawartość tej książki jest udostępniana w formie 'takiej, jaka jest', a autorzy i wydawcy nie udzielają żadnych gwarancji, wyrażonych ani dorozumianych, co do kompletności, dokładności, niezawodności, odpowiedniości lub dostępności informacji, produktów, usług lub związanych z nimi grafik zawartych w tej książce. Korzystanie z takich informacji odbywa się wyłącznie na własne ryzyko.

Autorzy i wydawcy nie ponoszą żadnej odpowiedzialności za jakiekolwiek straty lub szkody, w tym, między innymi, pośrednie lub wynikowe straty lub szkody, ani za jakiekolwiek straty danych lub zysków wynikające z utraty danych lub zysków w związku z korzystaniem z tej książki.

Ponadto, opisane w tej książce techniki i wskazówki są udostępniane wyłącznie w celach edukacyjnych i informacyjnych i nie powinny być wykorzystywane do jakichkolwiek nielegalnych lub szkodliwych działań. Autorzy i wydawcy nie popierają ani nie wspierają żadnych nielegalnych ani nieetycznych działań, a korzystanie z informacji zawartych w tej książce odbywa się na własne ryzyko i według własnego uznania użytkownika.

Użytkownik ponosi wyłączną odpowiedzialność za wszelkie działania podjęte na podstawie informacji zawartych w tej książce i powinien zawsze szukać profesjonalnej porady i pomocy przy próbie wdrożenia jakiejkolwiek z opisanych tu technik lub wskazówek.

Korzystając z tej książki, użytkownik zgadza się zwolnić autorów i wydawców od wszelkiej odpowiedzialności za jakiekolwiek szkody, straty lub szkody, które mogą wyniknąć z korzystania z tej książki lub jakichkolwiek informacji w niej zawartych.
{% endhint %}

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLAN SUBSKRYPCJI**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** mnie na **Twitterze** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów GitHub.

</details>
