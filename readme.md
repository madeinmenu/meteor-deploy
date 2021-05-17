# Déploiement Meteor sur EC2 (AWS)

> **Note:** éxécuter avec `sudo` si les commandes ne s'éxécutent pas ou si les permissions sont manquantes

## Meteor up

Si ce n'est pas déjà fait, installer Meteor Up en local

```sh
$ npm install --global mup
```

Ensuite, créer un dossier `.deploy` et faire un init

```sh
$ cd path/to/app
$ mkdir .deploy && cd .deploy
$ mup init
```

Configurer le fichier `mup.js`

```js
module.exports = {
  servers: {
    one: {
      host: 'x.x.x.x',
      username: 'username',
      pem: '/path/to/key.pem',
      // password: "",
    },
  },

  app: {
    name: 'appname',
    path: '/path/to/app/to/deploy',
    servers: {
      one: {},
    },
    buildOptions: {
      serverOnly: true,
    },
    env: {
      ROOT_URL: 'http://app.com',
      MONGO_URL: 'mongodb://localhost/meteor',
    },
    docker: {
      image: 'abernix/meteord:node-12-base',
    },
    // afficher la barre de progression d'upload du bundle
    enableUploadProgressBar: true,
  },
}
```

Une fois la configuration effectuée, ou à chaque fois qu'elle est changée

```sh
# --verbose pour afficher les tâches éxécutées
# mup.cmd sur windows
$ mup setup
```

Pour finalement déployer l'application

```sh
$ mup deploy
```

## Problèmes connus

### Le serveur distant n'a pas assez d'espace disque

<u>**Nettoyer les conteneurs docker de déploiement**</u>

```sh
# liste tous les conteneurs
$ docker ps -a

# arrête un conteneur actif
$ docker stop <containerId>

# supprime un conteneur
$ docker rm <containerId>
```

Et si le problème persiste

```sh
# force la suppression de tous les conteneurs inutilisés
$ docker system prune -af
```

<u>**Lister la taille des répertoires**</u>

Pour afficher la taille d'un répertoire (ou de l'espace disque global)

```sh
# -h affiche les unités en Ko, Mo et Go si nécéssaire
du -h <folderName>
```

exemple `du -h /tmp` qui va afficher la taille du répertoire des fichiers temporaires

<u>**Regarder si les fichiers temporaires ont été nettoyés**</u>

> /tmp is supposed to be cleaned up on reboot, but if you don't reboot (which is normal for servers), clean up will not happen

Pour nettoyer les fichiers inutilisés depuis plus de 10 jours

```sh
sudo find /tmp -type f -atime +10 -delete
```

### La connexion à mongoDB est échouée

Vérifier dans Atlas:

- si l'adresse IP publique de l'instance fait parti de la whitelist
- si l'adresse VPC CIDR (trouvable sur les détails du VPC AWS) est dans la whitelist (ex. x.x.x.x/16)

<img src="https://i.ibb.co/TBw5SJr/atlas1.png" width="80%">

### mongoDB retourne un code 14

Un processus Meteor est déjà en cours d'éxécution

```sh
# termine tous les processus Meteor
$ pkill -ef meteor
```
