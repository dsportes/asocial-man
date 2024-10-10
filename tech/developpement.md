---
layout: page
title: Les environnements de développement
---

Le développement a été réalisé avec le studio `VScode`.

Installations préliminaires requises:
- `node (npm)`, installé par `nvm`.
- `yarn`(plutôt que npm) gère les dépendances envers les modules externes.
- `webpack` est utilisé pour le packaging mais est installé directement dans les applications.

L'application Web est basée sur **Vue.js** (`vuejs.org`) avec la surcouche **Quasar** (`quasar.dev`).
- HTML et SASS (et non CSS),
- Javascript.
- `pinia` gère les _stores_.

Les _services_ sont du pur `Node.js` (Javascript), ainsi que l'utilitaire _upload_.

Le dépôt des applications est assuré sur `github.com`.

# Développement de l'application Web

Il s'effectue depuis le projet `asocial-app`.

L'installation de `quasar-cli` a une logique difficile à interpréter. Ci-dessous le processus sous Ubuntu 24.04 et yarn 4.4.1 :
- hors projet: `npm i -g quasar-cli`
- puis, une première fois dans le projet:

    npm install
    // puis enlever package.json.lock
    yarn install

    // ultérieurement on peut utiliser: yarn install

Le _mélange à éviter_ npm/yarn est apparu temporairement nécessaire. Les commandes directes `quasar ...` ne fonctionnent pas malgré l'installation _globale_ (il faut `yarn quasar ...` devant).

Lancement du serveur de test:

    yarn quasar dev -m pwa     // oui yarn ...

Build et test de la build:

    yarn quasar build -m pwa

    # https://github.com/http-party/http-server
    npx http-server dist/pwa -p 8080 --cors -S --cert ../asocial-srv/keys/fullchain.pem --key ../asocial-srv/keys/privkey.pem

    npx http-server dist/pwa -p 8080 --cors

L'application est une **PWA** (Progressive Web App) avec gestion d'un service_worker.

### Principaux modules requis
- `core-js` : vue3
- `animate.css`
- `msgpack` : sérialisation / désérialisation d'objets Javascript.
- `axios` : accès HTTP.
- `dexie` : accès aux bases IDB.
- `emoji-mart-vue-fast` : saisie d'emojis.
- `file-saver` : sauvegarde d'un fichier dans le browser.
- `mime2ext` : conversion de type MIME n code d'extensions de fichier.
- `vue-showdown` `github-markdown-css` : transformation d'un MD en HTML.
- `vue-advanced-cropper` : gestion locale d'une image (resize / crop).
- `pako` : compression / décompression GZIP de binaires.
- `simple-keyboard` : keyboard utilisable à la souris pour la saisie des phrases secrètes.
- `webcam-eaysy` : gestion de la webcam.
- `js-sha256` : hash SHA256 synchrone.
- `vue vue-i18n vue-router pinia` : modules de `vue`
- `quasar`

Lire le détail dans `package.json`

**Remarques pour i18n**. Dans `src/boot/i18.js` 
- ajouter `legacy: false,` sinon choix-langue ne s'initialise pas bien.
- importer `configStore` pour récupérer la valeur initiale de la locale.

### Fichier `quasar.config.js`
Les personnalisations majeures sont:
- `build.extendWebpack (cfg)` : règles pour le chargement des `.md .txt .svg`
- `devServer` : la configuration minimale (choix du port) pour le test.

#### Configuration de VScode pour le debug
Elle se fait dans `launch.json`. 
- Depuis VScode _>>> Run >>> Open configurations_

    {
    "version": "0.2.0",
    "configurations": [
        {
        "type": "chrome",
        "request": "launch",
        "name": "Quasar App: chrome",
        "url": "http://localhost:8081?doda",
        "webRoot": "${workspaceFolder}/src",
        "sourceMapPathOverrides": {
            "webpack://asocial/./src/*": "${webRoot}/*"
        }
        }
    ]
    }

Le serveur de test (qui fait un build local) se lance par: `yarn quasar dev -m pwa`

Voir le détail dans le document `applicationWeb.md`

# Développement des services

Le développement des services OP, PUBSUB, OP+PUBSUB et des _tools_ (export-db ...) se fait dans le même projet `asocial-srv`.

Les services sont à la base des serveurs HTTP même quand ils sont déployés en _cloud functions_. L'outil _tools_ est de facto une application locale nodejs.

### Principaux modules requis
- `@aws-sdk...` : 5 modules implémentant l'interface AWS S3.
- `@google-cloud...` : 3 modules requis pour accès à firestore, storage et d'interfaçage du logging Winston.
- `@open-wc/webpack-import-meta-loader` : Webpack loader for supporting import.meta in webpack.
- `msgpack` : sérialisation / désérialisation d'bjets Javascript.
- `js-sha256` : hash SHA256 synchrone. Invoqué depuis LE module commun à l'applicationWeb (qui ne dépend pas de nodejs) et les services.
- `express` : serveur HTTP.
- `better-sqlite3` : accès à la base SQLite.
- `node-args` : pour lire les arguments de la ligne de commande pour _tools_.
- `node-fetch` : pour le service OP quand il doit soumettre des requêtes à PUBSUB.
- `web-push` : envoi de notifications aux browsers des sessions Web.
- `winston` : gestionnaire de logs.
- `sendgrid/mail` : service d'envoi de mails _d'alerte_ par HTTP.

### Serveur de test
Depuis l'environnement de développement on peut lancer:
- `node src/server.js` : service OP ou OP+PUBSUB selon la configuration de `src/config.mjs`
- `node src/pubsub.js` : service PUBSUB (seul).
- `node src/tools.mjs ...` : outils de _tools_.
- `node src/gensecret.mjs` : cryptage du fichier confidentiel `keys.json` en `src/secret.mjs`

Selon la configuration `src/config.js` il faut mettre en place des folders / fichiers pour supporter la base de données et l'espace de stockage.

**En cas de test en HTTPS**, les deux fichiers `fullchain.pem privkey.pem` doivent être présents dans le folder `keys`.

#### Base de données SQLite
L'outil _DB Browser for SQLite_ est utilisé pour déclarer le schéma (voir le visualiser / modifier) et parcourir les données.

Le folder `sqlite` contient : 
- `schema.sql` : le schéma de la base qui peut être traité par DB Browser for SQLite.
- `delete.sql` : rest des données d'une base.
- `test.db3` : la base de données _courante_ de test.
- `test1.bk` : _backup_ #1 d'une base SQLite.

**Le mode WAL de SQLite**

Ce mode implique que deux fichiers `test.db3-shm` et `test.db3-wal` existent après exécution d'un test et sont requis.

La commende `./bk.sh 2` (extension `.cmd` en Windows) effectue  dans `sqlite` un backup de la base courante dans le fichier `test2.bk`. On peut ainsi avoir plusieurs _backups_ de base pris à des instants différents. Ce fichier est _propre_ et a intégré les transactions en cours dans le WAL.

La commande `./rst.sh 2` restaure le backup `test2.bk` sur la base courante et à ce moment il n'y a ni `-wal` ni `-shm`. A la limite le fichier `test.db3` pourrait, à cet instant, être stocké et diffusé.

#### Storage `fs` (file-system)
Le folder déclaré à la configuration (`fstorage` `fstorageb` ...) doit exister.

#### Storage `gc` (Google Cloud)
Il est géré par l'émulateur (voir ci-après) et n'a pas à être déclaré ailleurs. 
- les _backups_ exportés de l'émulateur sont stockés dans emulators.
- keys.mjs doit contenir l'entrée `service_account`. 

#### Storage `s3` (AWS S3)
En test installer `minio` (https://min.io/).
- son fichier de configuration est `minio.json`.
- sa _base_ de fichiers est localisée à l'installation de `minio` (typiquement `~minio/`).
- son _token_ est déclaré dans `keys.mjs` dans `s3_config`.

# Développement Firestore

Il y a une dualité entre Firebase et GCP (Google Cloud Platform):
- `firestore, storage, functions` sont _effectivement_ hébergés sur GCP.
- la console Firebase propose une vue et des fonctions plus simples d'accès mais moins complètes.
- il faut donc parfois retourner à la console GCP pour certaines opérations.

Consoles:

    https://console.cloud.google.com/
    https://console.firebase.google.com/

## CLI Firebase
https://firebase.google.com/docs/cli

Installation ou mise à jour de l'installation

    npm install -g firebase-tools

### Authentification

    firebase login

**MAIS ça ne suffit pas toujours,** il faut régulièrement se ré-authentifier:

    firebase login --reauth


### Delete ALL collections
Aide: firebase firestore:delete -

    firebase firestore:delete --all-collections -r -f

### Déploiement des index et rules
Les fichiers sont:
- `firestore.indexes.json`
- `firestore.rules`

    Déploiement (import)
    firebase deploy --only firestore:indexes

    Export des index dans firestore.indexes.json
    firebase firestore:indexes > firestore.indexes.EXP.json

### Emulator
Dans `src/config.mjs` remplir la section `env:`

    env: {
       // On utilise env pour EMULATOR
      STORAGE_EMULATOR_HOST: 'http://127.0.0.1:9199', // 'http://' est REQUIS
      FIRESTORE_EMULATOR_HOST: 'localhost:8080'
    },

Remarques:
- Pour storage: 
  - le nom de variable a changé au cours du temps. C'est bien STORAGE_...
  - il faut `http://` devant le host sinon il tente https
- Pour Firestore il choisit le port 8080. Conflit éventuel avec app par exemple.
- En cas de message `cannot determine the project_id ...`
  `export GOOGLE_CLOUD_PROJECT="asocial-test1"`

**Commandes usuelles:**

    Lancement avec mémoire vide:
    firebase emulators:start --project asocial-test1

    Lancement avec chargée depuis un import:
    firebase emulators:start --import=./emulators/bk1

    Le terminal reste ouvert. Arrêt par CTRL-C (la mémoire est perdue)

En cours d'exécution, on peut faire un export depuis un autre terminal:

    firebase emulators:export ./emulators/bk2 -f

**Consoles Web sur les données:**

    http://127.0.0.1:4000/firestore
    http://127.0.0.1:4000/storage

# Storage S3 : par min.io
`min.io` est une application serveur qui offre un service de _Storage_ selon le protocole S3 de AWS. 

Son usage _local_ permet de tester un accès AWS-S3, sachant qu'il existe plusieurs fournisseurs proposant ce service (pas seulement AWS).

Un objet de configuration, nommé par exemple `S3_config` a la syntaxe suivante:

    const S3_config = {
      credentials: {
        accessKeyId: 'access-asocial',
        secretAccessKey: 'secret-asocial'
      },
      endpoint: 'http://localhost:9000',
      region: 'us-east-1',
      forcePathStyle: true,
      signatureVersion: 'v4'
    }

Les clés `access` et `secret` sont données par le fournisseur d'accès choisi.

## Installation locale de min.io

    https://min.io/docs/minio/linux/index.html

    https://min.io/docs/minio/windows/index.html

Après download / install, se placer dans le répertoire choisi (ici `D:\minio`) et lancer le server :

    ./minio server start

Il s'affiche l'URL de la console et l'URL de l'API (celle du endpoint ci-avant). Ouvrir la console:
- Ouvrir `Access-Key` et créer les clés. Il est possible de donner un string en clair comme ci-dessus.
- Ouvrir `Buckets` et créer un bucket `asocial`.

On peut browser le contenu d'un bucket depuis la console.

# Création / gestion d'un `Google account`

(todo)

### Création d'un `service_account` Google
https://cloud.google.com/iam/docs/service-accounts-create

https://cloud.google.com/iam/docs/keys-create-delete#creating

Accéder au(x) `service_account` depuis la console
- `menu hamburger >> IAM & Admin >> Service Accounts`
- il y a 4 onglets. C'est l'onglet KEYS qui permet d'en générer un.
- Bouton `ADD KEY` et choisir `Create New Key`
- choisir le format JSON : ceci download le fichier.
- le sauvegarder en lieu sûr, il est impossible de le récupérer à nouveau.
  - on pourra seulement détruire cette clé,
  - en récréer une nouvelle comme décrit ci-dessus.

In fine, le json est intégré dans `src/keys.mjs` en syntaxe JS.

### `gcloud CLI` - CLI Google Cloud
https://cloud.google.com/sdk/docs/install


### demo-monprj
_Création_ au lancement de l'emulator

https://stackoverflow.com/questions/67781589/how-to-setup-a-firebase-demo-project


# Textes anciens à ignorer
### Ne pas utiliser ADC
Auth gcloud ADC: mais c'est "temporaire"
- gcloud auth application-default login

Revoke ADC:
- gcloud auth application-default revoke 

OU : ça marche aussi, différence avec au-dessus ? ca dure encore moins longtemps !!!!
- gcloud auth application-default login --impersonate-service-account daniel.sportes@gmail.com

    Linux, macOS: 
    $HOME/.config/gcloud/application_default_credentials.json
    
    Windows:
    %APPDATA%\gcloud\application_default_credentials.json

Déploiement gcloud:
- gcloud app deploy --verbosity debug
- gcloud app logs tail

Variables env:
export GOOGLE_APPLICATION_CREDENTIALS="$HOME/.config/gcloud/application_default_credentials.json"

export GOOGLE_APPLICATION_CREDENTIALS="$HOME/git/asocial-test1/config/service_account.json"

Powershell
$env:FIRESTORE_EMULATOR_HOST="[::1]:8680"
$env:FIRESTORE_EMULATOR_HOST="localhost:8080"

Linux
export FIRESTORE_EMULATOR_HOST="localhost:8080"