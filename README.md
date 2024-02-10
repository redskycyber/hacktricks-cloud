# HackTricks Cloud

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** με στο **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

<figure><img src=".gitbook/assets/cloud.gif" alt=""><figcaption></figcaption></figure>

_Λογότυπα και κίνηση Hacktricks σχεδιασμένα από_ [_@ppiernacho_](https://www.instagram.com/ppieranacho/)_._

{% hint style="success" %}
Καλώς ήλθατε στη σελίδα όπου θα βρείτε κάθε **κόλπο/τεχνική/οτιδήποτε σχετικό με CI/CD & Cloud** που έχω μάθει σε **CTFs**, **πραγματικά** περιβάλλοντα **εργασίας**, **έρευνα** και **ανάγνωση** ερευνών και ειδήσεων.
{% endhint %}

## **Μεθοδολογία Πεντεσταρίσματος CI/CD**

**Στην Μεθοδολογία CI/CD του HackTricks θα βρείτε πώς να πεντεστάρετε υποδομές που σχετίζονται με δραστηριότητες CI/CD.** Διαβάστε την ακόλουθη σελίδα για μια **εισαγωγή:**

{% content-ref url="pentesting-ci-cd/pentesting-ci-cd-methodology.md" %}
[pentesting-ci-cd-methodology.md](pentesting-ci-cd/pentesting-ci-cd-methodology.md)
{% endcontent-ref %}

## Μεθοδολογία Πεντεσταρίσματος Cloud

**Στην Μεθοδολογία Cloud του HackTricks θα βρείτε πώς να πεντεστάρετε περιβάλλοντα cloud.** Διαβάστε την ακόλουθη σελίδα για μια **εισαγωγή:**

{% content-ref url="pentesting-cloud/pentesting-cloud-methodology.md" %}
[pentesting-cloud-methodology.md](pentesting-cloud/pentesting-cloud-methodology.md)
{% endcontent-ref %}

## Άδεια

**Πνευματικά δικαιώματα © Carlos Polop 2023. Εκτός αν αναφέρεται διαφορετικά (οι εξωτερικές πληροφορίες που αντιγράφονται στο βιβλίο ανήκουν στους αρχικούς συγγραφείς), το κείμενο του** [**HACK TRICKS CLOUD**](https://github.com/carlospolop/hacktricks-cloud) **του Carlos Polop διατίθεται υπό την άδεια**[ **Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)**](https://creativecommons.org/licenses/by-nc/4.0/)**.**\
**Εάν θέλετε να το χρησιμοποιήσετε για εμπορικούς σκοπούς, επικοινωνήστε μαζί μου.**

## **Αποποίηση Ευθύνης**

{% hint style="danger" %}
Αυτό το βιβλίο, 'HackTricks Cloud,' προορίζεται μόνο για εκπαιδευτικούς και ενημερωτικούς σκοπούς. Το περιεχόμενο αυτού του βιβλίου παρέχεται ως έχει, και οι συγγραφείς και οι εκδότες δεν κάνουν καμία αναπαράσταση ή εγγύηση οποιουδήποτε είδους, ρητή ή σιωπηρή, σχετικά με την πληρότητα, την ακρίβεια, την αξιοπιστία, την καταλληλότητα ή τη διαθεσιμότητα των πληροφοριών, προϊόντων, υπηρεσιών ή γραφικών που περιέχονται σε αυτό το βιβλίο. Οποιαδήποτε εμπιστοσύνη που τοποθετείτε σε τέτοιες πληροφορίες είναι αυστηρά και μόνον με δική σας ευθύνη.

Οι συγγραφείς και οι εκδότες δεν φέρουν καμία ευθύνη για οποιαδήποτε απώλεια ή ζημία, συμπεριλαμβανομένης, χωρίς περιορισμό, έμμεσης ή αποθετικής απώλειας ή ζημίας, ή οποιασδήποτε απώλειας ή ζημίας που προκύπτει από απώλεια δεδομένων ή κερδών που προκύπτουν από, ή σε συνδεση με, τη χρήση αυτού του βιβλίου.

Επιπλέον, οι τεχνικές και οι συμβουλές που περιγράφονται σε αυτό το βιβλίο παρέχονται μόνο για εκπαιδευτικούς και ενημερωτικούς σκοπούς, και δεν πρέπει να χρησιμοποιούνται για παράνο
