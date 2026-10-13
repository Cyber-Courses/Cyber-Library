---
title: "Injection CQL"
description: "Une entrée non fiable concaténée dans des instructions Cassandra Query Language est analysée comme de la syntaxe, permettant à un attaquant de sortir de valeurs entre guillemets, d'ajouter des prédicats et d'élargir les lectures avec ALLOW FILTERING."
keywords:
  - injection CQL
  - Cassandra Query Language
  - injection de clause WHERE
  - ALLOW FILTERING
  - liaison de paramètres
---

# CQL

CQL (Cassandra Query Language) est la façon dont les applications lisent et écrivent dans un cluster Cassandra. Il prend en charge les **paramètres liés**, positionnels `?` ou nommés `:name`, que le pilote envoie hors du texte de l'instruction. Lorsque le code concatène plutôt une entrée utilisateur dans la chaîne de l'instruction, l'entrée est analysée comme du CQL et la requête devient injectable.

## Schéma vulnérable

```python
# user_id taken from the request
q = "SELECT * FROM users WHERE id = '" + user_id + "'"
session.execute(q)
```

La forme sûre lie une valeur (`WHERE id = %s` avec un tuple de paramètres) ; la forme injectable l'interpole directement dans le littéral entre guillemets.

## Ce que CQL vous donne et ne vous donne pas

CQL se lit comme SQL mais il est bien plus restreint, et les payloads qui l'ignorent ne font que lever des erreurs de syntaxe. Il n'y a **pas d'`UNION`**, pas de sous-requêtes, pas de jointures, et pas de `OR` arbitraire entre colonnes. Il n'y a pas de fonction sleep, benchmark ou de calcul lourd sur laquelle construire une inférence temporelle. Les primitives utilisables sont étroites et spécifiques :

- Fermer une valeur entre guillemets et ajouter de la syntaxe (voir [Injection de clause WHERE](where-clause-injection.md)).
- Assouplir ou ajouter des prédicats sur des colonnes que la requête filtre déjà.
- Ajouter `ALLOW FILTERING` pour échapper à la partition prévue et scanner toute la table (voir [Abus d'ALLOW FILTERING](allow-filtering-abuse.md)).
- Déduire des données selon que des lignes sont renvoyées (voir [Inférence en aveugle](blind.md)) ou à partir du texte d'erreur (voir [Par erreur](error-based.md)).

Une contrainte supplémentaire façonne chaque payload `WHERE` : Cassandra exige normalement que les colonnes de clé de partition soient contraintes, et rejette les filtres sur les colonnes non clés sauf si `ALLOW FILTERING` est présent. Cette règle est à la fois un obstacle et, délibérément détournée, un levier.

La séparation d'instructions avec `;` dépend du pilote. De nombreuses bibliothèques clientes rejettent plusieurs instructions dans un même `execute`, donc la surface fiable est une seule instruction réécrite, `BEGIN BATCH` étant le moyen de regrouper plusieurs écritures en une seule (voir [Injection d'instruction batch](../batch-statement-injection.md)).

Cette section couvre les mécaniques fondamentales du `WHERE`, l'abus d'`ALLOW FILTERING`, ainsi que l'inférence en aveugle et par erreur.

## Outils

- **cqlsh** : émettre et vérifier des payloads CQL contre le cluster.
- **Burp Suite** : intercepter et altérer les requêtes web adossées à CQL.

## Références

- [Apache Cassandra: CQL reference](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
