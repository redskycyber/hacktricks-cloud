# HackTricks Cloud

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 상품**](https://peass.creator-spring.com)을 구매하세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)를 **팔로우**하세요.
* **Hacking 트릭을 공유하려면** [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>

<figure><img src=".gitbook/assets/cloud.gif" alt=""><figcaption></figcaption></figure>

_Hacktricks 로고 및 모션 디자인 by_ [_@ppiernacho_](https://www.instagram.com/ppieranacho/)_._

{% hint style="success" %}
CTF, 실제 환경, 연구 및 뉴스를 통해 배운 **CI/CD 및 클라우드와 관련된 모든 해킹 트릭/기술/기타**를 찾을 수 있는 페이지에 오신 것을 환영합니다.
{% endhint %}

## **Pentesting CI/CD Methodology**

**HackTricks CI/CD Methodology에서는 CI/CD 활동과 관련된 인프라를 펜테스트하는 방법을 찾을 수 있습니다.** **소개를 위해 다음 페이지를 읽어보세요:**

{% content-ref url="pentesting-ci-cd/pentesting-ci-cd-methodology.md" %}
[pentesting-ci-cd-methodology.md](pentesting-ci-cd/pentesting-ci-cd-methodology.md)
{% endcontent-ref %}

## Pentesting Cloud Methodology

**HackTricks Cloud Methodology에서는 클라우드 환경을 펜테스트하는 방법을 찾을 수 있습니다.** **소개를 위해 다음 페이지를 읽어보세요:**

{% content-ref url="pentesting-cloud/pentesting-cloud-methodology.md" %}
[pentesting-cloud-methodology.md](pentesting-cloud/pentesting-cloud-methodology.md)
{% endcontent-ref %}

## License

**저작권 © Carlos Polop 2023. (책에 복사된 외부 정보는 원래 저작자에게 속합니다.)** [**HACK TRICKS CLOUD**](https://github.com/carlospolop/hacktricks-cloud) **의 텍스트는** [**Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**](https://creativecommons.org/licenses/by-nc/4.0/) **에 따라 Carlos Polop의 라이선스가 부여됩니다.**\
**상업적 목적으로 사용하려면 저에게 연락하세요.**

## **Disclaimer**

{% hint style="danger" %}
이 책 'HackTricks Cloud'는 교육 및 정보 제공 목적으로만 제공됩니다. 이 책의 내용은 '있는 그대로' 제공되며, 저자 및 출판사는 이 책에 포함된 정보, 제품, 서비스 또는 관련 그래픽의 완전성, 정확성, 신뢰성, 적합성 또는 가용성에 대해 명시적 또는 묵시적으로 어떠한 종류의 보증도하지 않습니다. 이러한 정보에 의존하는 경우 사용자의 책임으로 이루어집니다.

저자 및 출판사는 이 책의 사용으로 인해 발생하는 데이터 손실 또는 이익 손실을 포함한 어떠한 손실이나 손해에 대해서도 책임을 지지 않습니다.

또한, 이 책에 설명된 기술과 팁은 교육 및 정보 제공 목적으로만 제공되며, 불법적이거나 악의적인 활동에 사용해서는 안 됩니다. 저자 및 출판사는 불법적이거나 비윤리적인 활동을 지지하지 않으며, 이 책에 포함된 정보를 사용하는 것은 사용자의 자체적인 책임과 판단에 따라 이루어져야 합니다.

사용자는 이 책에 포함된 정보를 기반으로 한 모든 조치에 대해 전적으로 책임이 있으며, 이 책에 설명된 기술이나 팁을 구현하려는 경우 항상 전문적인 조언과 지원을 구해야 합니다.

이 책을 사용함으로써 사용자는 저자 및 출판사를 어떠한 책임과 책임도 면제하고, 이 책이나 그 안에 포함된 정보의 사용으로 인해 발생할 수 있는 어떠한 손해, 손실 또는 피해에 대해서도 책임을 지지 않는 데 동의합니다.
{% endhint %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 상품**](https://peass.creator-spring.com)을 구매하세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)를 **팔로우**하세요.
* **Hacking 트릭을 공유하려면** [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>
