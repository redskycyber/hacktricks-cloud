---
description: |
インフラストラクチャに関連するハッキングトリック/テクニック/その他のすべてを見つけるページへようこそ。これらは、私がCTF、実際の環境、研究、ニュースを読む中で学んだものです。
---

# HackTricks Cloud

<details>

<summary><strong>ハックトリックスをサポートして特典を得る！</strong></summary>

* **HackTricksで会社を宣伝したい**場合や、**最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロード**したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を手に入れる
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つける、私たちの独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションを
* **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)**または[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter**で私をフォローする🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**。**
* **ハッキングトリックを共有するには、PRを提出して** [**HackTricks**](https://github.com/carlospolop/hacktricks) **および** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **のGitHubリポジトリに参加してください。**

</details>

<figure><img src=".gitbook/assets/cloud gif.gif" alt="" width="563"><figcaption></figcaption></figure>

_Hacktricksのロゴとモーションデザインは_ [_@ppiernacho_](https://www.instagram.com/ppieranacho/)_ によって作成されました。_

{% hint style="info" %}
ここでは、**CI/CD＆クラウドに関連するハッキングトリック/テクニック/その他のすべて**を見つけることができます。これらは、**CTF**、**実際の**環境、**研究**、**研究とニュースの読書**で学んだものです。
{% endhint %}

## **Pentesting CI/CD Methodology**

**HackTricks CI/CD Methodologyでは、CI/CDアクティビティに関連するインフラストラクチャのペンテスト方法を見つけることができます。**以下のページを読んで、**導入**について説明します。

{% content-ref url="pentesting-ci-cd/pentesting-ci-cd-methodology.md" %}
[pentesting-ci-cd-methodology.md](pentesting-ci-cd/pentesting-ci-cd-methodology.md)
{% endcontent-ref %}

## Pentesting Cloud Methodology

**HackTricks Cloud Methodologyでは、クラウド環境のペンテスト方法を見つけることができます。**以下のページを読んで、**導入**について説明します。

{% content-ref url="pentesting-cloud/pentesting-cloud-methodology.md" %}
[pentesting-cloud-methodology.md](pentesting-cloud/pentesting-cloud-methodology.md)
{% endcontent-ref %}

## ライセンス

**Copyright © Carlos Polop 2023. それ以外の場所で特に指定されていない限り（書籍にコピーされた外部情報は元の著者に帰属します）、Carlos Polopによる**[**HACK TRICKS CLOUD**](https://github.com/carlospolop/hacktricks-cloud)**のテキストは**[**Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**](https://creativecommons.org/licenses/by-nc/4.0/)**の下でライセンスされています。**\
**商業目的で使用する場合は、お問い合わせください。**

## **免責事項**

{% hint style="danger" %}
この書籍『HackTricks Cloud』は、教育および情報提供の目的でのみ使用することを意図しています。この書籍の内容は「現状のまま」提供され、著者および出版者は、この書籍内の情報、製品、サービス、または関連するグラフィックの完全性、正確性、信頼性、適合性、または利用可能性について、明示または黙示を問わず、いかなる種類の表明または保証も行いません。したがって、そのような情報に依存することは、完全に自己の責任で行ってください。

著者および出版者は、この書籍の使用によって生じる、間接的または結果的な損失や損害、データの損失や利益の喪失を含む、いかなる損失や損害に対しても一切の責任を負いません。

さらに、この書籍で説明されている技術やヒントは、教育および情報提供の目的でのみ提供されるものであり、いかなる違法または悪意のある活動にも使用してはなりません。著者および出版者は、違法または非倫理的な活動を是認または支持するものではなく、この書籍内の情報の使用は、ユーザー自身のリスクと裁量に基づくものです。

ユーザーは、この書籍内の情報に基づいて行われるすべての行動について、自己の責任であり、これらの技術やヒントを実装しようとする場合は常に専門の助言と支援を求めるべきです。

この書籍を使用することで、ユーザーは、著者および出版者を、この書籍またはその中に含まれる情報の使用によって生じるいかなる損害、損失、または害に対しても、一切の責任と責任から免除することに同意します。
{% endhint %}

<details>

<summary><strong>ハックトリックスをサポートして特典を得る！</strong></summary>

* **HackTricksで会社を宣伝したい**場合や、**最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロード**したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を手に入れる
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つける、私たちの独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションを
* **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)**または[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter**で私をフォローする🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**。**
* **ハッキングトリックを共有するには、PRを提出して** [**HackTricks**](https://github.com/carlospolop/hacktricks) **および** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **のGitHubリポジトリに参加してください。**

</details>
