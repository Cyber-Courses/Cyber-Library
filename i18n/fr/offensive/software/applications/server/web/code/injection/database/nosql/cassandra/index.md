---
order: 1
title: "Injection Cassandra"
description: "Le CQL construit par chaînes de caractères contre Apache Cassandra permet à un attaquant de sortir de valeurs entre guillemets, d'ajouter des prédicats, de forcer des scans sur tout le cluster et d'introduire clandestinement des écritures supplémentaires dans un magasin wide-column."
keywords:
  - injection Cassandra
  - injection CQL
  - magasin wide-column
  - ALLOW FILTERING
  - injection d'instruction batch
---

# Cassandra

Apache Cassandra est un magasin wide-column distribué interrogé avec **CQL**, un langage similaire à SQL. CQL est injectable pour la même raison que SQL : lorsque le code applicatif concatène une entrée utilisateur dans une instruction au lieu de lier des paramètres (`?` ou `:name`), l'entrée est analysée comme de la syntaxe de requête plutôt que traitée comme une valeur. Un attaquant qui atteint un `WHERE` construit par chaînes peut fermer une valeur entre guillemets, assouplir des prédicats, ajouter `ALLOW FILTERING` pour échapper à la partition que la requête visait, et dans certains pilotes introduire clandestinement des écritures supplémentaires via un `BEGIN BATCH`.

CQL est délibérément étroit. Il n'a pas d'`UNION`, pas de sous-requêtes, pas de `OR` entre colonnes arbitraires, et pas de fonction sleep ou benchmark, donc les astuces relationnelles qui pilotent la SQLi classique ne s'appliquent pas ici. Les vraies primitives sont l'évasion de valeur entre guillemets, l'ajout de prédicats, et `ALLOW FILTERING` pour transformer une lecture de partition en un scan sur tout le cluster. Lorsque le magasin de données exécute des fonctions définies par l'utilisateur, un `CREATE FUNCTION` injecté peut atteindre l'exécution de code à l'intérieur de la JVM Cassandra. Ce sous-arbre couvre les mécaniques CQL, l'abus de batch et d'UDF, ainsi que l'inférence en aveugle et par erreur.

## Sous-thèmes

- **[CQL](cql/index.md)** : Une entrée non fiable concaténée dans des instructions Cassandra Query Language est analysée comme de la syntaxe, permettant à un attaquant de sortir de valeurs entre guillemets, d'ajouter des prédicats et d'élargir...

## Pages

- **[Injection d'instruction batch](batch-statement-injection.md)** : Injecter dans un bloc `BEGIN BATCH ... APPLY BATCH`, ou en former un, pour regrouper des écritures INSERT, UPDATE et DELETE supplémentaires dans une seule instruction Cassandra.
- **[Abus de fonction définie par l'utilisateur](user-defined-function-abuse.md)** : Lorsque les fonctions définies par l'utilisateur sont activées, un CREATE FUNCTION ou CREATE AGGREGATE injecté avec un corps Java exécute du code de l'attaquant à l'intérieur de la JVM Cassandra.

## Outils

- **cqlsh** : le shell CQL natif pour tester des payloads contre un cluster.
- **Burp Suite** : intercepter et manipuler les requêtes web adossées à CQL.

## Références

- [Apache Cassandra: CQL reference](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
