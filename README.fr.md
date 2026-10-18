# Lab DHCP et DNS avec dnsmasq : un lab de réseau pour la classe, dans une seule VM

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md)

[![check](https://github.com/mitsurutech/lab-dhcp-dns/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-dhcp-dns/actions/workflows/check.yml)

Un **lab DHCP et DNS** prêt à l’emploi pour enseigner comment un réseau distribue des adresses et répond à des noms, issu de la [plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning), où les écoles et les établissements donnent à chaque élève sa propre VM Ubuntu avec un terminal dans le navigateur. Chaque élève reçoit un petit réseau construit à l’intérieur de sa machine : un serveur et deux clients, `pc1` et `pc2`, qui démarrent **sans aucune adresse**. L’élève configure **dnsmasq**, le petit serveur DHCP et DNS que l’on trouve dans les box et les labs du monde entier, pour que le serveur attribue des adresses prises dans une plage, indique aux clients leur routeur et leur serveur DNS, donne à un client une **adresse fixe selon son adresse MAC**, et réponde pour les noms propres au lab. Rien n’est installé sur l’ordinateur de l’élève.

Le serveur et les clients sont des **espaces de noms réseau** sur un commutateur virtuel, et ce sont les vrais outils qui travaillent : `dnsmasq` sur le serveur, `dhclient` sur les clients, `dig`, `ip` et `tcpdump` pour observer. C’est tout l’intérêt.

## Le lab

| Lab | Fichier | Ce que fait l’élève | Page d’aide |
| --- | --- | --- | --- |
| DHCP et DNS | [`dhcp-dns.lab.yaml`](dhcp-dns.lab.yaml) | Écrire la configuration de dnsmasq pour que les clients reçoivent des adresses d’une plage avec le bon routeur et le bon serveur DNS, qu’un client obtienne toujours la même adresse, et que les deux noms du lab se résolvent sans qu’aucun autre ne le fasse | [Lab DHCP et DNS](https://mitsurutech.co.uk/fr/help/lab-dhcp-dns) |

Le lab laisse dans le dossier personnel de l’élève un README qui dit quoi faire et ne donne aucune des réponses. Il a une **vérification**, un script qui note une VM rendue sans que l’enseignant ait à la lire. Elle s’exécute sur une copie de la machine enregistrée après un démarrage à neuf, arrête le dnsmasq qui tournait, en démarre un à partir des fichiers enregistrés dans `/etc/dnsmasq.d`, efface les deux clients et les fait redemander. Elle ne réussit donc que si le comportement réel est le bon : les clients reçoivent des adresses de la plage avec le bon routeur, le bon serveur DNS et la bonne durée de bail, le second client obtient son adresse fixe, les deux noms se résolvent vers les bonnes adresses, et un nom inconnu n’obtient pas de réponse. Des adresses saisies à la main sur les clients, une réponse générique pour tous les noms, ou des noms placés dans `/etc/hosts` ne passent pas.

## Ce que contient le fichier

`dhcp-dns.lab.yaml` est une **définition de lab de VM** complète, exactement ce que la plateforme exécute :

| Clé | Ce que c’est |
| --- | --- |
| `cpus`, `memory`, `disk` | la taille de la VM de chaque élève : 1 CPU, 1024 Mo et 10 Go |
| `open` | les ports qui s’ouvrent dans le navigateur : aucun, le lab est le terminal |
| `setup` | un script exécuté une fois, en tant que root, au premier démarrage de la VM de chaque élève : il installe dnsmasq, le client DHCP, `dig` et `tcpdump` depuis le dépôt d’Ubuntu, écrit la commande `dnslab`, le README du lab et l’unité qui reconstruit le réseau au démarrage |
| `check` | le script de notation, exécuté en tant que root sur une copie du disque enregistré après un démarrage à neuf ; son code de sortie est le résultat et `0` signifie réussite |

La commande `dnslab` qu’il installe : `dnslab status` liste les machines, `dnslab restart` relance dnsmasq à partir de la configuration enregistrée, `dnslab renew pc1` fait demander un bail à un client et `dnslab shell pc1` ouvre un shell à l’intérieur d’une machine.

## L’utiliser sur la plateforme

**Pour le donner comme travail à une classe :**

1. Dans votre module, ajoutez une tâche et choisissez la carte **Lab de VM Ubuntu**.
2. Appuyez sur l’exemple, ou collez le contenu du fichier ci-dessus. Utiliser un exemple remplace le texte, et Annuler rend le vôtre.
3. Publiez et essayez d’abord votre propre copie. Chaque élève obtient une VM à lui, conservée d’une session à l’autre.
4. Distribuez-la comme n’importe quelle tâche. Quand elle est rendue, lancez la vérification sur la ligne d’un élève dans la liste de la classe : elle démarre une copie de sa machine enregistrée qui n’est pas réenregistrée, et indique si elle a réussi et ce qu’elle a affiché.

Le [guide des labs de VM Ubuntu](https://mitsurutech.co.uk/fr/help/labs) du manuel et sa [page des labs d’exemple](https://mitsurutech.co.uk/fr/help/sample-labs) couvrent le reste, et une leçon de base pour ce lab, **DHCP and DNS Basics**, se trouve dans le Public college et dans chaque nouvel établissement. Pour modifier le lab, copiez le fichier et modifiez ensemble les adresses et les noms dans `setup` et dans `check`.

Pas encore de compte d’établissement ? [La première année est gratuite](https://mitsurutech.co.uk/fr/pricing) jusqu’à 50 élèves, et [l’inscription](https://mitsurutech.co.uk/fr/signup) ne demande qu’une adresse e-mail.

## Vérifier le fichier

À chaque push, `.github/workflows/check.yml` vérifie que le fichier est un YAML valide avec les cinq clés ci-dessus et que `setup` et `check` tiennent chacun dans la limite de 16 Ko de la plateforme. Il ne démarre pas de VM : cela se fait sur les propres machines de la plateforme.

## Ce qui ne fonctionne pas dans un lab

La VM atteint le dépôt de paquets d’Ubuntu, PyPI, npm et GitHub, et rien d’autre : un lab ne peut donc pas être dirigé vers un hôte d’internet, et le serveur DNS du lab n’a nulle part où transmettre un nom. C’est une seule VM : le serveur et les clients sont des espaces de noms à l’intérieur, les clients ne sont donc pas des machines séparées, et il n’y a pas de second serveur DHCP avec lequel entrer en concurrence.

## À propos

Réalisé par [Mitsuru Technologies](https://mitsurutech.co.uk) pour la [plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning) : Python, Java, Go, Node, SQL et Linux dans le navigateur pour les écoles et les établissements, avec des labs de conteneurs, des labs de VM, des applications web déployées depuis GitHub et des quiz qui se corrigent seuls. Essayez du code sans compte dans le [bac à sable](https://mitsurutech.co.uk/fr/playground).

D’autres labs du même éditeur : [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea) et [lab-sample](https://github.com/mitsurutech/lab-sample).

## Licence

Cette œuvre est sous licence [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Vous pouvez la partager et l’adapter pour apprendre et enseigner, à condition de citer Mitsuru Technologies Ltd, de ne pas l’utiliser commercialement et de partager tout ce que vous en tirez sous la même licence. Le texte complet est dans le fichier `LICENSE`.

Les applications que ce lab installe gardent les licences de leurs propres projets : [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) est sous licence GPL-2.0 ou GPL-3.0 et le client DHCP d’ISC est sous licence MPL-2.0 ; tous deux sont installés depuis le dépôt d’Ubuntu au démarrage d’une VM, et rien d’eux n’est distribué ici.
