# HackTricks Cloud

<details>

<summary><strong>Erlernen Sie AWS-Hacking von Grund auf mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie mir auf **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) Github-Repositorys senden.

</details>

<figure><img src=".gitbook/assets/cloud.gif" alt=""><figcaption></figcaption></figure>

_Hacktricks-Logos & Motion-Design von_ [_@ppiernacho_](https://www.instagram.com/ppieranacho/)_._

{% hint style="success" %}
Willkommen auf der Seite, auf der Sie jeden **Hacking-Trick/Technik/Alles im Zusammenhang mit CI/CD & Cloud** finden, den ich in **CTFs**, **realen** Umgebungen, **Recherchen** und **Lesen** von Forschungen und Nachrichten gelernt habe.
{% endhint %}

## **Pentesting CI/CD-Methodik**

**In der HackTricks CI/CD-Methodik finden Sie, wie Sie die Infrastruktur im Zusammenhang mit CI/CD-Aktivitäten pentesten können.** Lesen Sie die folgende Seite für eine **Einführung:**

{% content-ref url="pentesting-ci-cd/pentesting-ci-cd-methodology.md" %}
[pentesting-ci-cd-methodology.md](pentesting-ci-cd/pentesting-ci-cd-methodology.md)
{% endcontent-ref %}

## Pentesting-Cloud-Methodik

**In der HackTricks Cloud-Methodik finden Sie, wie Sie Cloud-Umgebungen pentesten können.** Lesen Sie die folgende Seite für eine **Einführung:**

{% content-ref url="pentesting-cloud/pentesting-cloud-methodology.md" %}
[pentesting-cloud-methodology.md](pentesting-cloud/pentesting-cloud-methodology.md)
{% endcontent-ref %}

## Lizenz

**Copyright © Carlos Polop 2023. Sofern nicht anders angegeben (die externen Informationen, die in das Buch kopiert wurden, gehören den Originalautoren), ist der Text auf** [**HACK TRICKS CLOUD**](https://github.com/carlospolop/hacktricks-cloud) **von Carlos Polop lizenziert unter**[ **Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**](https://creativecommons.org/licenses/by-nc/4.0/)**.**\
**Wenn Sie es für kommerzielle Zwecke verwenden möchten, kontaktieren Sie mich.**

## **Haftungsausschluss**

{% hint style="danger" %}
Dieses Buch, 'HackTricks Cloud', dient ausschließlich zu Bildungs- und Informationszwecken. Der Inhalt dieses Buches wird auf einer 'wie es ist'-Basis bereitgestellt, und die Autoren und Verleger machen keine Darstellungen oder Gewährleistungen jeglicher Art, ausdrücklich oder implizit, über die Vollständigkeit, Richtigkeit, Zuverlässigkeit, Eignung oder Verfügbarkeit der Informationen, Produkte, Dienstleistungen oder Grafiken, die in diesem Buch enthalten sind. Jegliches Vertrauen, das Sie in solche Informationen setzen, erfolgt daher ausschließlich auf eigenes Risiko.

Die Autoren und Verleger haften in keinem Fall für Verluste oder Schäden, einschließlich, aber nicht beschränkt auf indirekte oder Folgeschäden oder Verluste oder Schäden jeglicher Art, die aus dem Verlust von Daten oder Gewinnen resultieren, die sich aus der Nutzung dieses Buches ergeben.

Darüber hinaus werden die in diesem Buch beschriebenen Techniken und Tipps ausschließlich zu Bildungs- und Informationszwecken bereitgestellt und sollten nicht für illegale oder bösartige Aktivitäten verwendet werden. Die Autoren und Verleger billigen oder unterstützen keine illegalen oder unethischen Aktivitäten, und die Verwendung der in diesem Buch enthaltenen Informationen erfolgt auf eigenes Risiko und Ermessen des Benutzers.

Der Benutzer ist allein verantwortlich für alle auf der Grundlage der in diesem Buch enthaltenen Informationen getroffenen Maßnahmen und sollte immer professionellen Rat und Unterstützung suchen, wenn er versucht, eine der hier beschriebenen Techniken oder Tipps umzusetzen.

Durch die Verwendung dieses Buches erklärt sich der Benutzer damit einverstanden, die Autoren und Verleger von jeglicher Haftung und Verantwortung für etwaige Schäden, Verluste oder Schäden freizustellen, die aus der Verwendung dieses Buches oder einer der darin enthaltenen Informationen resultieren können.
{% endhint %}

<details>

<summary><strong>Erlernen Sie AWS-Hacking von Grund auf mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie mir auf **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) Github-Repositorys senden.

</details>
