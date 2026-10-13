---
title: "Injection de fonctions show et list dans CouchDB"
description: "Les fonctions show et list de CouchDB transforment les documents avec du JavaScript côté serveur, donc une fonction injectée s'exécute pendant le rendu et peut lire au-delà des données prévues."
keywords:
  - fonction show CouchDB
  - fonction list CouchDB
  - rendu côté serveur
  - document de conception
  - provides
  - getRow
---

# Fonctions show et list

CouchDB peut rendre des documents dans des formats de sortie arbitraires côté serveur. Une fonction `show` transforme un document unique en une réponse HTTP ; une fonction `list` transforme les lignes d'une vue en une réponse. Les deux sont du JavaScript stocké dans un document de conception et exécuté par le query server au moment de la requête. Contrôler l'une ou l'autre fonction, ou le document de conception qui la contient, exécute du JavaScript de l'attaquant pendant le rendu.

## Fonctions show

Une fonction `show` prend un document et l'objet requête et renvoie un corps de réponse :

```json
{
  "shows": {
    "render": "function(doc, req){ return { body: JSON.stringify(doc), headers: { 'Content-Type': 'application/json' } }; }"
  }
}
```

Elle est invoquée contre un identifiant de document :

```http
GET /app/_design/pub/_show/render/user:42 HTTP/1.1
```

Un `show` injecté s'exécute avec la portée du query server. Parce que la fonction reçoit le `doc` entier, elle peut sérialiser des champs que l'application n'a jamais eu l'intention de publier, et parce qu'elle reçoit aussi `req`, elle peut se ramifier sur des paramètres de requête fournis par l'attaquant :

```javascript
function(doc, req){
  // req.query controlled by the caller
  return { body: JSON.stringify(doc[req.query.field]) };
}
```

Une requête telle que `_show/render/user:42?field=password_hash` renvoie alors le champ ciblé. L'objet `req` expose aussi les en-têtes, les cookies et le chemin brut, toutes des entrées influencées par l'attaquant sur lesquelles une fonction injectée peut agir.

## Fonctions list

Une fonction `list` s'exécute sur les lignes d'une vue, les tirant avec `getRow()` et diffusant la sortie :

```json
{
  "lists": {
    "dump": "function(head, req){ var row; while(row = getRow()){ send(JSON.stringify(row.value)); } }"
  }
}
```

Elle est invoquée comme un couple de document de conception, nom de list et nom de vue :

```http
GET /app/_design/pub/_list/dump/leak HTTP/1.1
```

Ici `leak` est une vue dans le même document de conception. Une fonction `list` injectée parcourt chaque ligne que la vue a produite et envoie chaque valeur à l'appelant, contournant toute pagination ou sélection de champs que l'application a superposée. Associer une fonction list permissive à une vue large (voir [Injection MapReduce](mapreduce.md)) diffuse l'ensemble complet en une seule réponse.

## La surface provides/registerType

Les fonctions show et list se ramifient couramment sur le format demandé via `provides()` et `registerType()` :

```javascript
function(doc, req){
  provides('html', function(){ return '<p>' + doc.name + '</p>'; });
  provides('json', function(){ return JSON.stringify(doc); });
}
```

Une fonction injectée peut enregistrer un gestionnaire qui émet du contenu choisi par l'attaquant, et parce que les corps de callback sont du JavaScript ordinaire, ils sont un point d'injection de code comme n'importe quel autre champ. Lorsqu'une application assemble un corps de show/list à partir d'une entrée utilisateur (un fragment de gabarit, une liste de champs), l'attaquant ferme l'expression environnante et ajoute ses propres instructions.

## Déclenchement et sondage

L'exécution des fonctions show et list est un simple `GET`, donc aucune écriture n'est nécessaire une fois la fonction existante. Pour confirmer un point d'injection lorsqu'une application construit ces fonctions dynamiquement, soumettez une entrée qui casse le JavaScript environnant et observez une erreur de compilation du query server, ou fournissez une branche inoffensive (`provides('txt', function(){ return 'marker'; })`) et demandez ce format pour voir le code injecté s'exécuter. Une fonction injectée qui lit `req` transforme l'endpoint de rendu en une primitive de lecture pilotée par paramètre sur les documents que l'appelant peut nommer.

## Outils

- **curl** : invoquer les endpoints _show et _list pour déclencher les fonctions rendues.
- **Burp Repeater** : forger des requêtes qui pilotent le rendu contrôlé par paramètre.

## Références

- [Apache CouchDB: Show functions](https://docs.couchdb.org/en/stable/ddocs/ddocs.html#show-functions)
- [Apache CouchDB: List functions](https://docs.couchdb.org/en/stable/ddocs/ddocs.html#list-functions)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
