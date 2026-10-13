---
order: 3
title: "Injection de clause WHERE : sortir des valeurs CQL entre guillemets"
description: "Fermer une valeur CQL entre guillemets et ajouter des prédicats permet à un attaquant d'élargir un filtre Cassandra, sous réserve de la contrainte de clé de partition que le WHERE impose normalement."
keywords:
  - injection WHERE CQL
  - évasion de valeur entre guillemets
  - contrainte de clé de partition
  - injection de prédicat
  - Cassandra
---

# Injection de clause WHERE

La valeur injectable atterrit le plus souvent à l'intérieur d'un littéral entre guillemets dans une clause `WHERE`. Un guillemet simple ferme ce littéral, et tout ce qui suit est analysé comme de la syntaxe CQL.

## Sortir du littéral

Étant donné le point d'injection :

```python
q = "SELECT * FROM users WHERE username = '" + name + "'"
```

une valeur de `' OR username = 'admin` réécrit l'instruction en :

```sql
SELECT * FROM users WHERE username = '' OR username = 'admin'
```

Le `'` final du gabarit ferme le littéral que l'attaquant a ouvert, laissant une syntaxe valide. CQL accepte les commentaires de ligne `--` et `//` ainsi que les commentaires de bloc `/* ... */`, donc n'importe lequel peut écarter le reste du gabarit :

```sql
' OR username = 'admin' /*
```

Les littéraux de chaîne utilisent des guillemets simples ; un guillemet simple littéral à l'intérieur d'une valeur est échappé en le doublant (`''`), ce qui compte lorsqu'un filtre supprime les guillemets bruts mais laisse passer la forme doublée jusqu'à l'analyseur.

## La contrainte de clé de partition

CQL ne se comporte pas comme SQL ici. Une table est partitionnée par sa clé de partition, et Cassandra **rejette** normalement un `WHERE` qui ne contraint pas la clé de partition complète, ou qui filtre sur une colonne non clé, sauf si la requête porte `ALLOW FILTERING`. Donc cette évasion classique :

```sql
SELECT * FROM users WHERE username = '' OR username = 'admin'
```

ne fonctionne que si `username` est la clé de partition. `OR` est lui-même restreint en CQL : ce n'est pas un opérateur général entre colonnes comme en SQL, donc les payloads de style `' OR 1=1 --` échouent généralement d'emblée. Une évasion qui cible une colonne non clé, ou qui élargit au-delà d'une seule partition, doit généralement ajouter `ALLOW FILTERING` pour être acceptée (voir [Abus d'ALLOW FILTERING](allow-filtering-abuse.md)).

## Ajouter et assouplir des prédicats

Lorsque la clé de partition est contrainte, vous pouvez quand même élargir le résultat à l'intérieur ou au travers de la clé en utilisant des opérateurs CQL que le moteur accepte :

```sql
' AND role IN ('admin','superuser') ALLOW FILTERING /*
' AND token(id) > token('') ALLOW FILTERING /*
```

`IN` énumère plusieurs valeurs ; `token()` permet de raisonner sur l'anneau. `CONTAINS` et `CONTAINS KEY` élargissent les colonnes collections :

```sql
' AND permissions CONTAINS 'admin' ALLOW FILTERING /*
```

Pour une colonne clé, fournir une valeur de partition différente pivote simplement vers une autre partition :

```sql
admin' /*
```

transforme une recherche pour le nom soumis en une recherche pour `admin`, le commentaire de bloc avalant la queue du gabarit.

## Sonder le point

Soumettez un `'` isolé et observez si une erreur d'analyse CQL remonte dans la réponse (`SyntaxException`, `line 1:...`), signal fiable que la valeur atteint le texte de la requête. Comparez une valeur inoffensive à `existing_value' /*` pour confirmer que le commentaire et l'évasion s'analysent correctement. Lorsqu'aucune ligne n'est renvoyée et que les erreurs sont supprimées, repliez-vous sur [Inférence en aveugle](blind.md) ; lorsque les erreurs fuient, voir [Par erreur](error-based.md).

## Outils

- **cqlsh** : tester les évasions de valeur entre guillemets et les ajouts de prédicats directement.
- **Burp Repeater** : forger et renvoyer le payload d'évasion via le paramètre.

## Références

- [Apache Cassandra: SELECT and the WHERE clause](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html#select-statement)
- [Apache Cassandra: Data definition and partition keys](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/ddl.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
