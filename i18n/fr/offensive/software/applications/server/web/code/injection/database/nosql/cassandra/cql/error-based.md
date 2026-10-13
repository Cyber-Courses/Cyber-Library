---
order: 2
title: "Par erreur : faire fuir le schéma Cassandra à travers les erreurs CQL"
description: "Les messages d'erreur CQL nomment les tables, les colonnes et les types attendus, permettant à un attaquant de cartographier un schéma Cassandra en provoquant délibérément des incompatibilités de type et des erreurs d'identifiant inconnu."
keywords:
  - injection CQL par erreur
  - divulgation de schéma Cassandra
  - incompatibilité de type
  - identifiant inconnu
  - InvalidRequest
---

# Par erreur

Lorsque les erreurs CQL se propagent jusqu'à la réponse, elles portent des détails de schéma et de type que l'application n'a jamais eu l'intention d'exposer. Un attaquant capable de déclencher des erreurs à la demande lit la structure directement depuis le texte du message au lieu de la déduire.

## Ce que les erreurs CQL révèlent

Cassandra rapporte des erreurs précises et structurées. Celles qui sont utiles pour cartographier un schéma :

- **`SyntaxException`** confirme que l'entrée atteint le texte de la requête et fait souvent écho au jeton fautif et à la position de ligne (`line 1:42 ...`), localisant où la valeur atterrit.
- **`InvalidRequest` / identifiant inconnu** nomme les colonnes et les tables. Référencer une colonne qui n'existe pas renvoie `Undefined column name <x>`, donc sonder des noms candidats énumère les vrais.
- Les erreurs d'**incompatibilité de type** indiquent le type CQL attendu. Comparer une colonne texte à un entier, ou inversement, renvoie un message nommant le type déclaré de la colonne.

## Provoquer une incompatibilité de type

Si une valeur est comparée à une colonne, forcer une comparaison incompatible fait signaler au moteur le type de la colonne. Étant donné un point d'injection de type texte, injecter une comparaison avec un entier :

```sql
' AND age = 'not_a_number' ALLOW FILTERING /*
```

contre une colonne `int` produit un message du genre `Expected 4 or 0 byte int` ou `Unable to make int from 'not_a_number'`, confirmant qu'`age` est un entier. Inverser l'incompatibilité confirme une colonne texte :

```sql
' AND username = 12345 ALLOW FILTERING /*
```

renvoyant `Expected 8 or 0 byte long` ou un message de type invalide qui nomme le type déclaré.

## Énumérer colonnes et tables

Les erreurs d'identifiant inconnu transforment les suppositions en confirmations. Chaque nom candidat soit s'analyse correctement (la colonne existe), soit renvoie `Undefined column name` :

```sql
' AND password = 'x' ALLOW FILTERING /*
' AND password_hash = 'x' ALLOW FILTERING /*
' AND is_admin = true ALLOW FILTERING /*
```

Un nom qui ne produit pas d'erreur est une vraie colonne ; un nom qui renvoie le message de colonne indéfinie ne l'est pas. La même technique contre les noms de tables, lorsque le point d'injection permet d'atteindre la cible du `FROM`, confirme l'existence de la table.

## Lire le keyspace système

Lorsque l'injection peut atteindre une cible `SELECT` complète plutôt qu'une simple valeur de `WHERE`, le keyspace `system_schema` contient le catalogue, et le texte d'erreur lors de son sondage fait encore fuir des détails même lorsque les lignes ne sont pas renvoyées :

```sql
SELECT column_name, type FROM system_schema.columns WHERE keyspace_name = 'app' AND table_name = 'users'
```

`system_schema.tables`, `system_schema.columns` et `system_schema.keyspaces` décrivent chaque keyspace, table, colonne et type. Combinées à l'oracle d'erreur ci-dessus, elles permettent à un attaquant de reconstruire le schéma avant de passer à l'extraction de données via [Inférence en aveugle](blind.md).

## Garder les erreurs visibles

L'extraction par erreur dépend de la survie des messages jusqu'à la réponse. Soumettre un `'` isolé pour confirmer qu'une `SyntaxException` est renvoyée est la première vérification ; s'il est avalé, le canal est booléen uniquement. Lorsque les messages sont renvoyés, ils sont le moyen le plus rapide de cartographier la structure avant de commencer la récupération des données.

## Outils

- **cqlsh** : reproduire les erreurs d'incompatibilité de type et d'identifiant inconnu pour lire le schéma.
- **Burp Repeater** : soumettre des payloads provoquant des erreurs et inspecter les messages renvoyés.

## Références

- [Apache Cassandra: system_schema tables](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html)
- [Apache Cassandra: CQL data types](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/types.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
