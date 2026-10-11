---
title: "Plateformes de chat d'équipe : attaquer les serveurs de chat auto-hébergés via leur API REST et leurs intégrations"
description: "Le chat d'équipe auto-hébergé (Mattermost, Rocket.Chat) est une application web plus une API REST et un websocket au-dessus d'un stockage Postgres ou Mongo. La surface d'attaque comprend l'authentification de l'API, l'autorisation défaillante (IDOR entre équipes et salons), l'injection dans la couche d'authentification et de requêtes, et les fonctionnalités d'intégration, de webhook et de plugin réservées aux administrateurs qui atteignent l'exécution de code sur le serveur. Couvre l'identification du produit et de la version et où aller ensuite."
keywords:
  - team chat
  - mattermost
  - rocket.chat
  - chat server api
  - integration rce
---

# Plateformes de chat d'équipe

Un produit de chat d'équipe auto-hébergé, ce sont trois services derrière un seul nom d'hôte : une application web qui rend le client, une **API REST** qui fait tout le vrai travail, et un **websocket** qui diffuse les événements en direct, le tout adossé à une base de données relationnelle ou documentaire (Postgres ou MySQL pour Mattermost, MongoDB pour Rocket.Chat). Presque toutes les primitives offensives résident dans l'API REST plutôt que dans l'interface du navigateur, car l'interface n'est qu'un client parmi d'autres de cette API et l'API est atteignable directement avec `curl`. Cette forme pose les quatre questions récurrentes : pouvez-vous vous authentifier ou abuser de la couche d'authentification, pouvez-vous atteindre les données d'autres équipes et salons via une autorisation défaillante (IDOR), pouvez-vous injecter dans la couche de requêtes ou de connexion, et pouvez-vous atteindre la machinerie d'intégration, de webhook ou de plugin réservée aux administrateurs qui exécute du code attaquant sur le serveur.

La propriété décisive est que les deux produits livrent de puissantes fonctionnalités d'extension côté serveur aux administrateurs : Mattermost charge les plugins Go téléversés dans le processus serveur, et Rocket.Chat exécute des « scripts » d'intégration dans un bac à sable qui a historiquement pu être évadé vers `require`/`process`. Ainsi, l'état final d'une compromission de serveur de chat est rarement « lire quelques messages » ; c'est l'exécution de code en tant que compte de service une fois que vous détenez un jeton admin, et la couche d'authentification est là où ce jeton se gagne.

Ce domaine concerne uniquement les produits auto-hébergés (l'application web plus son API REST sur votre cible), pas les clouds SaaS Slack, Teams ou Discord.

## Triage : identifier le produit et la version

Un seul balayage non authentifié sépare les deux produits et cerne la version, ce qui décide quel contournement d'authentification et quel chemin d'intégration sont actifs :

```bash
# Mattermost: Go server, default 8065, REST under /api/v4
curl -sk http://<target>:8065/api/v4/system/ping
#   {"status":"OK"} (and an X-Version-Id response header on most routes)
curl -skI http://<target>:8065/api/v4/users | grep -i x-version-id
#   X-Version-Id: 9.5.0.9.5.0.<hash>.true   -> exact build
curl -sk 'http://<target>:8065/api/v4/config/client?format=old' | tr ',' '\n' | grep -i version
#   "Version":"9.5.0","BuildNumber":...     unauthenticated client config

# Rocket.Chat: Node/Meteor, default 3000, REST under /api/v1
curl -sk http://<target>:3000/api/info
#   {"version":"6.5.0","success":true}      unauthenticated version
curl -skI http://<target>:3000/sockjs/info  # Meteor/SockJS transport confirms Rocket.Chat/Meteor
```

Lisez les signaux : un `/api/v4/system/ping` renvoyant `{"status":"OK"}` et un en-tête `X-Version-Id`, c'est Mattermost ; une `version` JSON de `/api/info` plus un transport DDP `/sockjs/` et l'en-tête `X-Instance-Id`, c'est Rocket.Chat. Le chemin des ressources statiques (`/static/` pour le bundle servi par Go de Mattermost) et le balisage de connexion le confirment. Enregistrez la version exacte avant de toucher à la couche d'authentification, car le contournement de connexion NoSQL et les chaînes de plugins/intégrations dépendent de la version.

## Sous-thèmes

- **[Mattermost](mattermost/index.md)** : serveur Go avec l'API REST `/api/v4` au-dessus de Postgres/MySQL. Énumération via `/api/v4/users` et `/api/v4/teams`, la couche d'authentification par connexion et jeton d'accès personnel, et l'exploitation serveur via IDOR d'API, injection SQL par paramètre de filtre, et upload de plugin Go signé/non signé vers l'exécution de code.
- **[Rocket.Chat](rocket-chat/index.md)** : serveur Node/Meteor avec l'API REST `/api/v1` et DDP au-dessus de MongoDB. Énumération via `users.list`/`spotlight`, le contournement de connexion par injection d'opérateur NoSQL, et l'exploitation serveur en chaînant ce contournement vers un jeton admin et l'évasion du bac à sable de script d'intégration pour la RCE.

## Références

- [Mattermost API reference (`/api/v4`)](https://api.mattermost.com/)
- [Rocket.Chat REST API reference (`/api/v1`)](https://developer.rocket.chat/apidocs)
- [HackTricks: pentesting network services](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
- [OWASP Testing Guide: testing for NoSQL injection](https://owasp.org/www-project-web-security-testing-guide/)
