---
title: "Rocket.Chat : attaquer le serveur REST /api/v1 et ses scripts d'intégration"
description: "Attaquer un serveur Rocket.Chat auto-hébergé : un processus Node/Meteor sur 3000 exposant l'API REST /api/v1 et DDP au-dessus de MongoDB. Identifiez le build depuis /api/info et le transport Meteor /sockjs/, puis progressez à travers l'énumération des utilisateurs et des canaux, l'injection NoSQL qui prend le contrôle d'un compte admin, et l'exploitation du serveur en l'enchaînant à un jeton admin et à l'évasion de la sandbox des scripts d'intégration pour l'exécution de code."
keywords:
  - rocket.chat
  - rocket.chat api
  - nosql injection login
  - integration script rce
  - meteor ddp
---

# Rocket.Chat

Rocket.Chat est une application Node.js bâtie sur le framework Meteor, servant un client web et une API REST enracinée à `/api/v1` sur le port `3000` par défaut (couramment mandaté vers 443), avec le protocole DDP de Meteor sur un websocket `/sockjs/` et MongoDB en dessous. Le socle MongoDB et la gestion permissive des corps de requête par Meteor pilotent l'attaque signature : l'endpoint de connexion accepte un objet structuré là où une chaîne est attendue, de sorte qu'un opérateur de requête Mongo injecté dans `user` ou `password` transforme l'authentification en une requête qui correspond sans le vrai mot de passe. Détenez n'importe quel jeton admin et la fonctionnalité **integrations** réservée aux admins exécute du JavaScript côté serveur dans une sandbox dont on a pu s'échapper à répétition vers `require`/`process`, ce qui est la voie vers l'exécution de code en tant qu'utilisateur Node.

## Identifier d'abord

Confirmez le produit et épinglez le build, car le comportement du contournement de connexion et la sandbox des scripts d'intégration diffèrent selon la version.

```bash
curl -sk http://<target>:3000/api/info
#   {"version":"6.5.0","success":true}       confirms Rocket.Chat + exact version
curl -skI http://<target>:3000/sockjs/info   # Meteor/SockJS transport (DDP) present
curl -skI http://<target>:3000/ | grep -i x-instance-id   # X-Instance-Id header is Rocket.Chat
```

`/api/info` donne la version sans authentification sur la plupart des builds ; là où c'est verrouillé, les données de bootstrap Meteor dans la source de la page et le blob `__meteor_runtime_config__` divulguent tout de même la version. Notez si `/api/v1/settings.public` est accessible sans authentification (ci-dessous) car il expose si l'inscription est ouverte et quelles méthodes de connexion sont activées. Enregistrez la version avant d'attaquer la couche de connexion.

## Pages

- **[Énumération](enumeration.md)** : récolter les utilisateurs et les canaux via `/api/v1/users.list`, `/api/v1/channels.list` et `/api/v1/spotlight`, lire `settings.public`, et utiliser les méthodes DDP Meteor ; interpréter le JSON renvoyé avec les en-têtes `X-Auth-Token`/`X-User-Id`.
- **[Authentification](authentication.md)** : pourquoi le `/api/v1/login` REST résiste à un objet opérateur naïf, l'injection NoSQL en aveugle dans les méthodes de compte et de réinitialisation qui extrait un jeton de réinitialisation de mot de passe stocké pour prendre le contrôle d'un admin, et le contournement par recherche de nom d'utilisateur `ddp-streamer` Enterprise.
- **[Exploitation du serveur](server-exploitation.md)** : enchaîner le contournement d'authentification à un jeton admin, puis l'évasion de la sandbox des « scripts » d'intégration entrants/sortants et des webhooks vers `require`/`process` pour l'exécution de code, le point de sortie d'upload de fichier, et la SSRF du parseur de messages ; aboutissant à un shell en tant qu'utilisateur node.

## Références

- [Rocket.Chat REST API reference (`/api/v1`)](https://developer.rocket.chat/apidocs)
- [Rocket.Chat integrations and script documentation](https://docs.rocket.chat/docs/integrations)
- [RocketChat/Rocket.Chat (server source)](https://github.com/RocketChat/Rocket.Chat)
- [Meteor DDP protocol specification](https://github.com/meteor/meteor/blob/devel/packages/ddp/DDP.md)
