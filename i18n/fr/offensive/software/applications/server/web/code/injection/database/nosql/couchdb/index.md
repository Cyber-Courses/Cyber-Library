---
order: 2
title: "Injection CouchDB"
description: "Apache CouchDB est une base de données documentaire HTTP/JSON dont les documents de conception exécutent du JavaScript côté serveur, donc l'injection cible ces fonctions ainsi que l'API de requête, de vue et de configuration."
keywords:
  - injection CouchDB
  - document de conception
  - query server
  - MapReduce
  - fonctions show et list
  - injection de requête de vue
---

# CouchDB

Apache CouchDB est une base de données documentaire avec une API REST HTTP et des documents JSON. Chaque opération est un verbe HTTP contre une URL : `GET /db/doc`, `PUT /db/_design/app`, `POST /db/_find`. Ce qui fait de CouchDB une cible d'injection distincte, c'est que ses documents de conception portent du **JavaScript** qui s'exécute côté serveur dans un processus query server : fonctions map et reduce, transformations `show` et `list`, gestionnaires `update`, et filtres `validate_doc_update`.

L'injection prend ici deux formes. La première cible la surface JavaScript : lorsqu'un attaquant peut écrire un document de conception, ou atteindre la configuration `_config`/`query_servers`, du code contrôlé par l'attaquant s'exécute à l'intérieur du serveur. La seconde cible l'API de requête et de vue : les paramètres de clé, de plage et `include_docs` passés dans les requêtes `_all_docs`, `_find` et `_view` peuvent être manipulés pour lire des documents hors de la portée prévue.

## Pages

- **[Injection JavaScript](javascript.md)** : Écrire un document de conception ou atteindre les réglages _config query_servers exécute du JavaScript de l'attaquant dans le processus query server de CouchDB, atteignant l'exécution de code.
- **[Injection MapReduce](mapreduce.md)** : Les fonctions map et reduce de CouchDB sont du JavaScript, donc une définition de vue contrôlée par l'attaquant s'exécute côté serveur et emit() peut faire fuir des documents hors de la portée prévue...
- **[Fonctions show et list](show-list-function.md)** : Les fonctions show et list de CouchDB transforment les documents avec du JavaScript côté serveur, donc une fonction injectée s'exécute pendant le rendu et peut lire au-delà des données prévues...
- **[Injection de requête de vue](view-query.md)** : Manipuler les paramètres key, startkey, endkey, include_docs et _all_docs sur les requêtes de vue CouchDB permet de lire des documents hors de la plage de clés prévue.

## Outils

- **curl** : envoyer des requêtes HTTP/JSON brutes à l'API REST de CouchDB.
- **Burp Suite** : intercepter et altérer les requêtes de document et de vue.

## Références

- [Apache CouchDB: HTTP API reference](https://docs.couchdb.org/en/stable/api/index.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
