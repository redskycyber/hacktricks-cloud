# HackTricks Cloud

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> ile sıfırdan kahraman olmak için AWS hackleme öğrenin<strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **reklamını görmek** veya HackTricks'i **PDF olarak indirmek** için [**ABONELİK PLANLARINA**](https://github.com/sponsors/carlospolop) göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* Özel [**NFT'lerden**](https://opensea.io/collection/the-peass-family) oluşan koleksiyonumuz olan [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)'ı **takip edin**.
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna **PR göndererek** paylaşın.

</details>

<figure><img src=".gitbook/assets/cloud.gif" alt=""><figcaption></figcaption></figure>

_Hacktricks logoları ve hareketleri_ [_@ppiernacho_](https://www.instagram.com/ppieranacho/)_ tarafından tasarlandı._

{% hint style="success" %}
CTF'lerde, gerçek hayatta, araştırma yaparken ve haberleri okurken öğrendiğim **her CI/CD & Cloud ile ilgili hacking hilesi/teknik/ne varsa** bulacağınız sayfaya hoş geldiniz.
{% endhint %}

## **Pentesting CI/CD Metodolojisi**

**HackTricks CI/CD Metodolojisinde, CI/CD faaliyetleriyle ilgili altyapıyı pentest etmenin nasıl yapılacağını bulacaksınız.** Bir **giriş** için aşağıdaki sayfayı okuyun:

{% content-ref url="pentesting-ci-cd/pentesting-ci-cd-methodology.md" %}
[pentesting-ci-cd-methodology.md](pentesting-ci-cd/pentesting-ci-cd-methodology.md)
{% endcontent-ref %}

## Cloud Pentesting Metodolojisi

**HackTricks Cloud Metodolojisinde, bulut ortamlarını pentest etmenin nasıl yapılacağını bulacaksınız.** Bir **giriş** için aşağıdaki sayfayı okuyun:

{% content-ref url="pentesting-cloud/pentesting-cloud-methodology.md" %}
[pentesting-cloud-methodology.md](pentesting-cloud/pentesting-cloud-methodology.md)
{% endcontent-ref %}

## Lisans

**Telif Hakkı © Carlos Polop 2023. Başka bir şekilde belirtilmediği sürece (kitaba kopyalanan harici bilgilerin orijinal yazarlara ait olduğu), Carlos Polop'un** [**HACK TRICKS CLOUD**](https://github.com/carlospolop/hacktricks-cloud) **adlı kitabındaki metin**[ **Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**](https://creativecommons.org/licenses/by-nc/4.0/)**** **lisansı altındadır.**\
**Ticari amaçlarla kullanmak isterseniz, benimle iletişime geçin.**

## **Reddi Beyan**

{% hint style="danger" %}
Bu kitap, 'HackTricks Cloud,' sadece eğitim ve bilgilendirme amaçlıdır. Bu kitaptaki içerik 'olduğu gibi' temelinde sunulmaktadır ve yazarlar ve yayıncılar, bu kitap içindeki bilgilerin, ürünlerin, hizmetlerin veya ilgili grafiklerin eksiksizlik, doğruluk, güvenilirlik, uygunluk veya kullanılabilirlik konusunda herhangi bir temsil veya garanti yapmamaktadır. Bu bilgilere dayanarak yaptığınız herhangi bir güven tamamen kendi sorumluluğunuzdadır.

Yazarlar ve yayıncılar, bu kitabın kullanımıyla ilgili olarak, dolaylı veya ardışık kayıp veya hasar dahil olmak üzere, veri kaybı veya kar kaybı gibi herhangi bir kayıp veya hasardan sorumlu tutulamazlar. Bu kitabın içindeki bilgilerin kullanımından kaynaklanan herhangi bir zarar, kayıp veya zarardan kaynaklanan herhangi bir sorumluluktan yazarlar ve yayıncılar sorumlu tutulamazlar.

Ayrıca, bu kitapta tanımlanan teknikler ve ipuçları sadece eğitim ve bilgilendirme amaçlıdır ve herhangi bir yasa dışı veya kötü niyetli faaliyet için kullanılmamalıdır. Yazarlar ve yayıncılar, herhangi bir yasa dışı veya etik olmayan faaliyeti onaylamaz veya desteklemez ve bu kitapta yer alan bilgilerin kullanımı, kullanıcının kendi riski ve takdirine tabidir.

Kullanıcı, bu kitapta yer alan bilgilere dayalı olarak yaptığı herhangi bir eylemden tamamen sorumludur ve burada tarif edilen tekniklerin veya ipuçlarının herhangi birini uygulamaya çalışırken her zaman profesyonel tavsiye ve yardım aramalıdır.

Bu kitabı kullanarak, kullanıcı, yazarları ve yayıncıları bu kitabın kullanımı veya içindeki bilgiler nedeniyle ortaya çıkabilecek herhangi bir zarar, kayıp veya zarardan muaf tutmayı kabul eder.
{% endhint %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> ile sıfırdan kahraman olmak için AWS hackleme öğrenin<strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **reklamını görmek** veya HackTricks'i **PDF olarak indirmek** için [**ABONELİK PLANLARINA**](https://github.com/sponsors/carlospolop) göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* Özel [**NFT'lerden**](https://opensea.io/collection/the-peass-family) oluşan koleksiyonumuz olan [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)'ı **takip edin**.
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna **PR göndererek** paylaşın.

</details>
