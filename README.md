# HackTricks Cloud

<details>

<summary><strong>AWSハッキングをゼロからヒーローまで学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksにあなたの会社を広告したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見する、私たちの独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクション
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**テレグラムグループ**](https://t.me/peass)に**参加する**、または**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)で**フォローする**。
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出して、あなたのハッキングのコツを共有する。

</details>

<figure><img src=".gitbook/assets/cloud.gif" alt=""><figcaption></figcaption></figure>

_Hacktricksのロゴとモーションは_ [_@ppiernacho_](https://www.instagram.com/ppieranacho/) _によってデザインされました。_

{% hint style="success" %}
CTF、実際の環境、研究、および研究とニュースの読書で学んだ**CI/CD & Cloudに関連するハッキングのコツ/テクニック/その他**を見つけるページへようこそ。
{% endhint %}

## **Pentesting CI/CD Methodology**

**HackTricks CI/CD Methodologyでは、CI/CD活動に関連するインフラストラクチャのペネトレーションテスト方法を見つけることができます。** 以下のページを読んで**導入部分をご覧ください:**

{% content-ref url="pentesting-ci-cd/pentesting-ci-cd-methodology.md" %}
[pentesting-ci-cd-methodology.md](pentesting-ci-cd/pentesting-ci-cd-methodology.md)
{% endcontent-ref %}

## Pentesting Cloud Methodology

**HackTricks Cloud Methodologyでは、クラウド環境のペネトレーションテスト方法を見つけることができます。** 以下のページを読んで**導入部分をご覧ください:**

{% content-ref url="pentesting-cloud/pentesting-cloud-methodology.md" %}
[pentesting-cloud-methodology.md](pentesting-cloud-methodology.md)
{% endcontent-ref %}

## ライセンス

**著作権 © Carlos Polop 2023。特に指定されている場合を除き（本にコピーされた外部情報は元の著者に属します）、[**HACK TRICKS CLOUD**](https://github.com/carlospolop/hacktricks-cloud)上のテキストは、Carlos Polopによって**[**Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**](https://creativecommons.org/licenses/by-nc/4.0/)**ライセンスの下にあります。**\
**商用目的で使用したい場合は、私に連絡してください。**

## **免責事項**

{% hint style="danger" %}
この本「HackTricks Cloud」は、教育および情報提供の目的のみを意図しています。この本の内容は「現状のまま」で提供され、著者および出版者は、この本に含まれる情報、製品、サービス、関連するグラフィックの完全性、正確性、信頼性、適合性、または利用可能性について、明示的または暗黙的ないかなる表現も保証もしません。したがって、そのような情報に依存することは、完全にあなた自身のリスクになります。

著者および出版者は、この本の使用から生じる、またはそれに関連して生じるいかなる損失または損害、特に間接的または結果的な損失または損害、またはデータの損失または利益の損失に起因するいかなる損失または損害についても、いかなる場合においても責任を負いません。

さらに、この本に記載されているテクニックやヒントは、教育および情報提供の目的のみのために提供されており、違法または悪意のある活動には使用されるべきではありません。著者および出版者は、違法または非倫理的な活動を支持または支援しません。この本に含まれる情報の使用は、ユーザーの自己責任と裁量において行われます。

ユーザーは、この本に含まれる情報に基づいて行われるいかなる行動についても、完全に責任を負います。また、ここに記載されているテクニックやヒントを実装しようとする際は、常に専門家のアドバイスと支援を求めるべきです。

この本を使用することにより、ユーザーは、この本またはその中に含まれる情報の使用から生じる可能性のあるいかなる損害、損失、または害について、著者および出版者を一切の責任および責任から解放することに同意します。
{% endhint %}

<details>

<summary><strong>AWSハッキングをゼロからヒーローまで学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksにあなたの会社を広告したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見する、私たちの独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクション
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**テレグラムグループ**](https://t.me/peass)に**参加する**、または**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)で**フォローする**。
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出して、あなたのハッキングのコツを共有する。

</details>
