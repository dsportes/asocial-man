---
layout: page
title: Analyse de la confiance dans l'application en exécution
---

> Cette note comporte un mix _d'éléments humains_ et de _considérations techniques_. Selon la _culture_ de chacun, elle peut apparaître comme une collection d'évidences ou de discours abscons.

## Confiance humaine
Au delà de la pétition de principe que in fine tout aboutit à une confiance humaine, le plus important est d'analyser ce qui se passe si elle est prise en défaut.

On peut essayer de consacrer des moyens considérables à essayer de casser une clé de cryptage, mais le fait est que la corruption et la menace physique sur son propriétaire est en général un procédé moins coûteux pour obtenir le même résultat.

### Une structure non hiérarchique qui _confine_ les conséquences des fuites des _utilisateurs_
**Dans les structures _dictatoriales_, un _sommet_ détient plus d'information que les niveaux inférieurs**. Mais la structure de l'application **n'est pas** hiérarchique mais à plat, en réseau:
- _faire tomber_ la confidentialité d'un compte ne fait tomber qu'une partie de la confidentialité:
  - celles des notes du compte,
  - celles des _groupes_ dont le compte est membre, notes et / ou liste des membres. Si les notes révèlent un contenu, la liste des membres ne révèlent que des cartes de visite où le propriétaire a mis ce qu'il voulait (et rarement son adresse postale et son identité réelle).
  - celles des chats avec d'autres comptes, dont là encore le contenu de l'échange est connu et la carte de visite peut-être non fiable.
- il est impossible d'imposer aux comptes de toujours inclure un responsable hiérarchique dans leurs groupes, sauf par une menace forte au cas où un compte récalcitrant se ferait prendre -ce qui reste aléatoire-.

En conséquence il n'y pas de moyens pour faire tomber en masse les comptes d'un espace en _tapant sur une tête_: la perte de confidentialité est toujours restreinte ce qui **confine** les risques humains d'infiltration / corruption / menace.

### Les _agents_ techniques directs et de second niveau
Plusieurs _agents techniques_ interviennent à différents niveaux, les conséquences de leurs défaillances sont analysés dans la suite de ce document:
- **le développeur** : il a écrit le logiciel, est responsable du contenu de son _source_ initial, pas des transformations et extensions qui peuvent se produire ensuite.
- **l'agent de déploiement de l'application Web** : depuis le source de l'application Web il a configuré les adresses des services centraux à accéder et a _déployé_ sur un site Web l'application pour qu'elle puisse être invoquée par les utilisateurs.
- **l'administrateur technique** a effectué le travail équivalent pour déployer sur un _serveur_ (pour simplifier) l'application _serveur_ (les deux services des _opérations_ et de _push web_).

Les _agents de second niveau_ sont, en vue simplifiée, ceux des _providers_ de service externes:
- _provider du site Web_ d'où l'application Web peut être invoquée.
- _provider du serveur central_, ou des _cloud functions_ correspondantes.
- _provider de la base de données_ responsable d'assurer la confidentialité des données qui y sont stockées.
- _provider du storage_ ou les fichiers attachés aux notes sont stockés.

Ce qui est analysé ci-dessous n'est pas la _fiabilité / disponibilité_ de leurs prestations mais leur capacité à assurer la _confidentialité_ vis à vis des autres bien entendu mais aussi d'eux-mêmes. Plus prosaïquement, le risque zéro n'existant pas,
- comment déceler une éventuelle faille de confidentialité ?
- que se passe-t-il, quelles conséquences, une faille (toujours possible) de confidentialité a-t-elle ?

## Confiance technique de base
Cette confiance est celle que nous sommes obligés d'avoir dans certains éléments techniques, faute de quoi rien n'est possible. Quelques uns sont listés ci-dessus.

### Le couple _matériel / Operating System + browser_ des terminaux
Un terminal peut avoir des _portes dérobées_ qui par exemple,
- trace les frappes au clavier,
- liste des données en mémoire de l'application en cours d'exécution, comme un debugger à distance non contrôlé par le propriétaire du poste.

Sans confiance dans ces matériels / logiciels aucune confidentialité n'est possible à assurer: les textes à afficher sont _en clair_ dans la mémoire des postes sinon ... ils ne peuvent pas s'afficher, les clés de cryptage sont aussi en clair sinon ... elles ne peuvent ni crypter ni décrypter.

> L'application Web permet de saisir sa phrase secrète sur un _clavier_ applicatif: ceci empêche d'intercepter les frappes au clavier depuis un _clavier hacké_. Si c'est la mémoire du _browser_ qui est accessible depuis l'extérieur, certes la saisie n'est pas interceptée mais son résultat l'est.

> De la paranoïa ? Peut-être mais des mobiles piratés à distance ont fait l'actualité et la rumeur dit que les géants du Web ont leur propre version de Linux dans leurs serveurs pour éviter d'être hackés à leur insu.

### Le couple _matériel / Operating System_ des serveurs
Quand le serveur est hébergé par un _provider_ externe, on en revient à la question plus générale d'avoir confiance en lui ou non.

Quand on est son propre hébergeur, la confidentialité est compromise si le _provider de la VM_ (pour simplifier) a placé des _portes dérobées_ permettant de lire les données en cours d'exécution dans les processus exécutant Node.

## Conséquences des compromissions selon les _niveaux / agents techniques_
### _Données humaines_ versus _meta-données_
**Les _données humaines_** sont les textes et fichiers humainement interprétables,
- les textes des notes, des chats, des cartes de visite et leurs photos,
- par extension les clés de cryptage permettant de les crypter / décrypter.

Les _identifiants_ des comptes, avatars, notes, chats, groupes ... sont des chaînes de 12 chiffres / lettres latines minuscules et majuscules aléatoires. Un identifiant ne _signifie_ rien tout seul. 

**Les _meta-données_ sont des _relations_ entre identifiants**:
- liste des avatars d'un compte,
- liste des groupes accédés par un avatar / compte,
- liste des invitations aux groupes d'un avatar / compte,
- liste des comptes sponsorisés par un compte A en attente de réponse,
- liste des _contacts_ d'un avatar / compte,
- liste des tickets de crédits d'un compte (leur numéros, pas leur contenu).

Par extension sont des _meta-données_,
- les compteurs d'abonnement / consommation d'un compte,
- la date de fin de validité d'un compte,
- les _numéros de version_, le nombre de fois où un document (compte, avatar, note ...) a été modifié.

### Règles majeures
**Aucune _donnée humaine_ ne sort en clair de la mémoire d'une session** en cours d'exécution sur un _terminal_.
- même ce qui s'écrit sur la _micro base de données locale_ est crypté par la clé majeure du compte.

Autrement dit à la double condition,
- que le terminal soit _techniquement digne de confiance_,
- que le logiciel de l'application Web qui s'y exécute n'ait pas subi de transformation depuis son source,

il est impossible de lire aucune _données humaines_ sur aucun autre support technique que la mémoire d'une session.

**Les _meta-données_ ne sont lisibles en clair QUE dans la mémoire des serveurs en cours d’exécution**:
- les sessions en exécution dans un browser n'ont en mémoire en clair QUE les meta-données relatives au périmètre du compte auxquels elles sont connectés.

**La base données** n'expose pas les _meta-données_, elles sont cryptées par la même clé qui permet à un serveur de les interpréter dans sa mémoire.

### Conséquences sur le détournement de la base de données
- Les _meta-données_ n'y sont pas lisibles en clair: il faut en faire un extrait et les décrypter par la clé de l'administrateur technique.
- Les _données humaines_ sont inaccessibles, cryptées par les données des comptes.

### Conséquences sur le détournement de l'espace de _storage_ des fichiers
Il n'y a (quasiment) pas de _meta-donnée_ stockées. Les contenus des fichiers sont cryptés par des clés que seuls les comptes connaissent. 
- On ne peut seulement qu'en tirer le nombre et le volume des fichiers par _identifiant_ d'avatar ou de groupe.

# Problématique de confiance
On suppose que le _matériel / Operating System_ est de confiance.

On suppose aussi que les algorithmes de hash et de cryptage sont fiables:
- SHA-256 et PBKFD.
- AES-256.
- RSA-2048.

Aucune information _publique_ ne permet aujourd'hui d'affirmer le contraire, du moins jusqu'à, peut-être, une diffusion massive de calculateurs quantiques.

### Logiciels _open source_
Tous les logiciels utilisés par l'application sont _open source_, leurs sources peuvent être lus et analysés librement.

Ceci s'arrête aux frontières suivantes:
- certains browsers sur les terminaux: _Chrome_ n'est pas open source.
- l'exécutable de Node sur les serveurs.
- certains logiciels de base de données.

> **Ce n'est pas parce que le logiciel de l'application est open source que pour autant c'est celui-là qui s'exécute !**

A partir de ces prémisses la _confiance_ globale repose sur les points suivants:
- **l'application Web: celle en exécution est-elle celle originale ?**
- **l'application Serveur: celle en exécution est-elle celle originale ?**
- **la clé de cryptage de l'administrateur technique sur les serveurs est-elle restée secrète ?**

> Si les réponses à ces 3 questions sont OUI, la confiance dans l'application est acquise: les réponses dépendent de chaque choix de déploiement. 

# L'application Web en exécution est-elle celle originale ?

Depuis les sources d'origine, **l'agent de déploiement de l'application Web**,
- peut ajuster certaines constantes de configuration dans le fichier de configuration.
- doit procéder à un _build_, une _pseudo compilation_ pour rendre le code distribuable.
- doit ajouter hors du code distribuable un fichier donnant les URLs des serveurs et de la documentation.
- doit installer le tout sur un serveur Web à disposition des utilisateurs.

En théorie **l'agent de déploiement de l'application Web** doit documenter et rendre _public aux utilisateurs_ les paramètres de son déploiement avec:
- le numéro de révision du source original,
- le fichier de _config_ éventuellement patché (voire quelques autres, comme les dictionnaires le cas échéant),
- le fichier des URLs des serveurs.

Normalement une équipe de _certification_,
- est capable de reproduire ce build et d'obtenir les quelques (gros) fichiers qui en résultent. 
- peut obtenir ces mêmes fichiers depuis l'URL de l'application,
- peut les comparer.

> Il est **possible** à une équipe de _certification_ externe de s'assurer que la version mise en ligne est conforme aux sources d'origine, que les patchs sont licites et que l'application Web redirige bien vers les _vrais_ serveurs et non vers des _pirates_.

> Remarque: ceci suppose que l'opération de _build_ par la même version de _webpack_ est _idem potente_, lancée deux fois avec les mêmes entrées elle produit exactement les mêmes sorties, ce qui est loin d'être prouvé.

**Pour l'application Web la _confiance_ peut reposer sur deux entités qui s'ignorent, l'une certifiant le travail de l'autre, c'est bien in fine une confiance _humaine organisationnelle_.**

> Cette confiance est capitale: la confidentialité des textes et clés repose sur ce niveau.

# L'application Serveur: celle en exécution est-elle celle originale ?

**L'administrateur technique** a quelques opérations à exécuter pour déployer l'application _Serveur_ depuis les sources _certifiées_ disponibles sur git:
- ajuster la configuration dans le fichier _config_,
- ajouter les _données secrètes_ qui ne sont pas dans git: tokens d'authentification vis à vis des providers, et quelques autres dont la clé de cryptage des données dans la base.
- selon le cas, effectuer un _build_,
- déployer le résultat chez son provider _Serveur_.

**Le résultat de cette séquence est invérifiable:**
- une équipe de _certification_ ne peut pas obtenir le résultat déployé chez le _provider serveur_,
- elle ne pourrait pas s'assurer que le code initial n'a pas été patché à des fins répréhensibles.

**Il est impossible de s'assurer en exécution que le code qui s'exécute est bien celui qui a été déployé:** il faut faire confiance au provider du serveur. Techniquement un service qui s'exécute ne renvoie pas sur demande,
- ni la signature du container Node qui s'exécute,
- ni la signature du code déployé (simplement parce qu'il n'est pas signé).

#### Les _providers_ ayant pignon sur rue sont-ils de confiance ?
Clairement Google, Microsoft, Amazon Web Services ne s'amusent pas, ni à déformer le code distribué, ni à ouvrir des portes dérobées permettant de patcher / lire / tracer la mémoire des process en exécution.
- techniquement ils peuvent le faire,
- sur demande _insistante_ d'une autorité de sécurité nationale, le feraient-ils ?

### Alternative: assurer soi-même le rôle de provider _serveur_
C'est toujours possible et on peut toujours avoir son propre cloud privé où les VMs sont _safe_.

La question de confiance est ramenée à celle dans l'équipe d'administration ... constituée d'humains corruptibles et pouvant subir des pressions, mais où un protocole de vérification humain est possible.

# La clé de cryptage de l'administrateur technique sur les serveurs est-elle restée secrète ?

Cette clé permet à un intervenant extérieur ayant accès en lecture à la base de données, d'en lire les données dans le champ crypté qui contient les _meta-données_.

Le _provider_ d'accès à la base de données est-il en mesure de lire tous les champs de la base ?
- bien entendu, sinon il ne pourrait pas gérer la base,
- les colonnes / attributs _cryptés_ ne lui sont accessibles qu'en binaire crypté et il n'en a pas la clé. Il ne peut rien en faire et donc ne peut pas techniquement rendre visible les _meta-données_.

> Remarque: l'administrateur technique gère la clé de cryptage et le jeton d'accès à la base. S'il détourne l'une il peut aussi détourner l'autre.

**L'administrateur technique** est en mesure technique de diffuser ces clés à un _hacker_ qui sera en mesure d'accéder à la base et d'extraire / diffuser les _meta-données_.

# Conclusion
**Les _données humaines_ ne sont accessibles que dans la mémoire des sessions en exécution de l'application Web.**
- c'est celle qui _peut_ être le plus facilement certifiée par une équipe autonome de celle l'ayant déployée.
- cette équipe _peut_ s'assurer assez rapidement que le code déployé n'est pas malicieux, ni ne dirige vers des serveurs malicieux.

**Il existe des moyens techniques et organisationnels pour s'assurer que les _données humaines_ sont effectivement protégées.**

**Mais seule la confiance dans une chaîne organisationnelle permet d'estimer que le code qui s'exécute sur le serveur n'est pas malicieux**. Cette même chaîne doit aussi s'assurer de la confidentialité du fichier donnant la clé de cryptage en base de donnes.

**Si cette chaîne est jugée efficace, les _meta-données_ sont aussi protégées.**

Si ce n'est pas le cas, au pire les _meta-données_ peuvent fuiter: on n'en tire pas grand-chose, seulement des relations entre identifiants aléatoires anonymes qui ne peuvent être vraiment exploités que s'ils sont rapprochés des personnes physiques et morales dans le vrai monde.
