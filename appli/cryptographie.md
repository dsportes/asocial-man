---
layout: page
title: Quelques principes de cryptographie
---

**Tous les textes humainement intelligibles, images de cartes de visite, fichiers sont cryptées sur l'appareil de l'utilisateur** et ne transitent jamais en clair sur le réseau, ni ne sont stockés nulle part en clair.
- les fichiers sont stockés dans leur _Storage_ cryptés par la clé de la note à laquelle ils sont attachés (clé d'un groupe ou clé K d'un compte).

**Toutes les clés de cryptage pour chaque compte sont cryptées sur l'appareil de l'utilisateur** par la clé majeure K du compte ou la clé RSA privée de ses avatars, elles-mêmes cryptées par la phrase secrète du compte.
- Aucune clé de cryptage n'est transmise en clair au serveur, ni ne transite en clair sur le réseau, ni n'est stockée en clair sur disque.

> **Remarque:** l'inviolabilité des algorithmes cités ne considèrent pas les calculateurs quantiques, dont à ce jour ils ne semblent pas y avoir de disponibilité commerciale pour casser ces codes.

## Technologies de cryptage / hash employées
### Hash: SHA256
Ce hash d'une suite de bytes à une longueur de 32 bytes.

Il n'a pas été publié de cas où deux entrées différentes produisaient le même SHA256 (ce qui mathématiquement est certes possible).

SHA256 a un avantage qui est aussi un inconvénient: il est rapide à calculer et l'usage de processeurs graphiques accélèrent son calcul.
- tout est relatif: une attaque par force brute (essai de toutes les combinaisons possibles) pour tenter de retrouver le texte source depuis son SHA256 devient impraticable pour des sources d'une longueur De 20 bytes et au-delà, l'énergie consommée en calcul devenant inaccessible.
- il n'en reste pas moins que SHA256 n'est pas approprié pour hacher des mots de passe / phrase secrète.

### Hash : PBKFD
Ce hash d'une suite de bytes à une longueur de 32 bytes.

Il n'a pas été publié de cas où deux entrées différentes produisaient le même PBKFD (ce qui mathématiquement est certes possible).

Le PBKFD est _long_, volontairement, et son algorithme n'est pas accéléré par l'usage de processeurs graphiques.

PBKFD est utilisé pour hacher les phrases secrètes et de rencontre / sponsoring:
- avec une source imposée à 24 signes au minimum, l'attaque par force brute est impossible.
- mais casser un mot de passe se pratique aussi en utilisant des _dictionnaires_ de mots usuels. 

Une phrase secrète de 24 signes qui évite les répétitions d'un code court, qui parsème le texte de séparateurs en chiffres, etc. ne sera pas trouvée par usage de dictionnaires.

### Hash court
C'est seulement un repliement d'un SHA256 sur 9 bytes, soit 12 caractères chiffres ou lettres minuscules / majuscules.

### Cryptage symétrique AES256
La même clé de 32 bytes est employée pour crypter et décrypter un texte.
- une autre clé dite _SALT_ est employée pour compliquer le piratage: il faut avoir la clé et le _SALT_ pour décrypter un texte.
- les _SALT_ sont usuellement dans le code.
- le cryptage est rapide et les textes cryptés peuvent avoir n'importe quel longueur.

Il n'a pas été rendu public qu'un texte source (long) ait pu être retrouvé par application de force brute sur une clé de cryptage aléatoire de 32 bytes.

Le premier byte d'une chaîne cryptée est le numéro du _SALT_ employé au cryptage:
- il suffit donc que les logiciels cryptant / décryptant utilisent la même liste de _SALT_.
- en tirant au hasard ce premier byte, une même source cryptée deux fois aura des cryptages différents 255 fois sur 256: ainsi la comparaison entre deux chaînes cryptées n'indiquent pas si la source est la même ou non.
- toutefois quand une chaîne cryptée sert d'identifiant externe, il faut à l'inverse que la même source donne toujours le même résultat crypté: ceci s'obtient en forçant le cryptage à utiliser le _SALT_ premier de la liste plutôt qu'un aléatoire.

### Cryptage asymétrique RSA2048
Le cryptage nécessite une paire de clés générées ensemble:
- la clé publique sert à crypter un texte qui ne pourra être décrypté qu'en utilisant la clé privéE. Les deux clé sont longues.
- le texte _source_ a une longueur maximale de 256 bytes.
- le cryptage est lent.
- deux cryptages successifs d'un même texte source donne deux cryptages différents.

> 2048, c'est 256 bytes. Attaquer par force brute une telle clé semble irréaliste.

L'objectif est que le _public_ puisse librement crypter un texte à destination d'un seul destinataire.

Quand le texte à crypter peut être plus long que 256 bytes, on génère une clé symétrique aléatoire qui crypte le long texte et on crypte cette clé par la clé RSA.

Chaque avatar a un couple de clés RSA, tout un chacun ayant accès à la la clé publique peut crypter un texte que seul l'avatar cible peut décoder.

# Les clés d'un compte

## Phrase secrète
Quand un compte définit sa phrase secrète on s'assure que celle-ci n'a pas déjà été employée en utilisant son PBKFD. 

MAIS, donner à quelqu'un l'information que la phrase a déjà été employée, c'est lui donner l'assurance qu'il peut accéder à un autre compte !.

Pour chaque phrase on calcule deux PBKFD,
- celui de phrase complète,
- celui de ses 12 premiers signes.
- on exclut la possibilité de choisir une phrase secrète dont le PBKFD de la phrase raccourcie a déjà été employée.

Certes le compte sait ainsi qu'un autres compte a une phrase secrète dont le début est le même que la sienne, mais ses tentatives pour trouver la fin sont d'un coût rédhibitoire, d'autant plus qu'il n'en connaît pas la longueur.

## Clé "K" d'un compte
**C'est une clé de 32 bytes tirée au hasard**: il est impensable d'essayer de la retrouver par force brute.

Elle est stockée dans le document majeur du compte cryptée par le PBKFD de la phrase secrète:
- toutes les autres clés générées sont stockées cryptées par cette clé.
- on peut changer de _phrase secrète_ facilement: le changement re-crypte la clé K mais ne la change pas.

> La clé K n'est jamais communiquée en dehors d'une session. Elle figure en clair dans la session du compte puisqu'elle sert en permanence à obtenir d'autres clés. Mais le compte connaît SA _phrase secrète_ ... fouiller la mémoire pour trouver la clé K n'a aucun intérêt.

La base locale d'un compte sur un poste est une base de type _clé / valeur_.
Les clés comme les valeurs sont cryptées par la clé K:
- celle-ci est rangée dans cette base cryptée par le PBKFD de la phrase secrète.
- pirater une base locale ne sert à rien: soit on n'a pas la clé et on ne la trouvera pas, soit on l'a ... et on a pas besoin de pirater des données auxquelles on peut accéder en clair par l'application.

La clé K crypte toutes les notes des avatars du compte et ses fichiers.

## Clé "CV" d'un avatar
Quand un avatar est créé, il est générée une clé dite CV qui crypte sa carte de visite. La clé CV est mémorisée dans le document maître de l'avatar cryptée par la clé K du compte.

Cette clé est aussi incassable que les autres sauf qu'elle va être donnée à tous les autres avatars _en contact_, ceux à qui justement on a accordé le droit de lire sa carte de visite.

> La **fragilité** n'est pas dans le procédé cryptographique mais dans le nombre de _contacts_ à qui on a attribué sa confiance.

Cela dit le risque se limite à ce que des contacts pas forcément désirables un jour puissent lire votre carte de visite. Les autres données sont définitivement inaccessibles aux autres avatars.

## Clé d'un chat entre deux avatars
Elle est générée aléatoirement à la création du chat et est cryptée par les clés K respectives des deux comptes. A la création, le créateur du chat crypte cette clé par la clé publique RSA de l'autre: il n'y a que lui qui puisse la décoder, par sa clé privée encryptée par la clé K encryptée par la phrase secrète.

## Clé d'un groupe
Elle est générée aléatoirement à la création du groupe.
- chaque membre du groupe la reçoit lors de son invitation cryptée par sa clé RSA publique.
- il ré-encrypte cette clé par sa clé K plus tard dans une session ouverte.

La clé d'un groupe crypte aussi sa carte de visite.

La clé du groupe crypte toutes les notes du groupe et leurs fichiers attachés.

> La **fragilité** n'est pas dans le procédé cryptographique mais dans le nombre de _membres_ à qui le groupe a attribué sa confiance.

## Clé d'un sponsoring
Un sponsoring est créé par un avatar pour être lu par ... une personne qui n'a pas de compte. Il est donc impossible d'utiliser une clé RSA qu'il na pas encore.

On utilise le PBKFD de la phrase de sponsoring comme clé de cryptage:
- seul le destinataire la connaît (du moins c'est ce qui est conseillé),
- le document de sponsoring ne vit pas longtemps et il est inutilisable une fois qu'il a servi à créer un compte sponsorisé.

## Données NON cryptées
Certaines données ne sont pas cryptées tout le temps. Par exemple un Ticket de crédit:
- son identifiant est généré aléatoirement.
- le Comptable accède à ces tickets _en clair_ mais est incapable de savoir qui a généré ce ticket.
- le compte émetteur a ses tickets cryptés par sa clé K.

## Identifiants des documents
Ce sont des string de **12 lettres majuscules ou minuscules et chiffres**, issus d'un hash d'une clé aléatoire de 32 bytes (sauf l'identifiant du Comptable). Le premier signe indique le type de document identifié (avatar, groupe, note ...).

### Clé et identifiant d'un avatar
La **clé de la carte de visite** d'un avatar est constituée de 32 octets (256 bits) tirés au sort à la création de l'avatar. Elle crypte sa _carte de visite_.

Son identifiant est basé sur le _hash_ de sa clé.

Par exception, l'identifiant du Comptable est `300000000000` et la clé de sa carte de visite n'est pas secrète ... vu qu'il n'a pas de carte de visite.

### Identifiant d'un compte
Un compte et son avatar principal ont le même identifiant.

# Données cryptées en base de données
Certaines données ne peuvent pas être cryptées en base de données parce qu'elles servent de clé d'accès ou d'index.

Pour chaque type de document quelques données sont donc _en clair_:
- l'identifiant (un terme ou deux termes): mais c'est un string aléatoire non porteur d'information.
- des dates de péremption ... qui porte sur des documents dont on ne sait pas à qui appartiennent.
- des numéros de versions de 1 à N.

Les autres propriétés des documents sont scellées encodées dans un texte binaire crypté par la _clé d'administration du site_ que l'administrateur a _caché_ avec quelques rares autres données confidentielles comme les _token_ d'accès aux services d'accès externes (base de données, storage ...).

Les services du serveur peuvent évidemment lire ces propriétés des documents MAIS PAS le fournisseur d'accès des bases de données. Par exemple si on utilise une base de données de Google (Firestore), Google peut lire les index -à supposer que ça l'intéresse- mais aucune des autres informations qui sont encapsulées dans des binaires cryptés dont il n'a pas la clé.
