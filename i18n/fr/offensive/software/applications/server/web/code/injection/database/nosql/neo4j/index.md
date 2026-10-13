---
order: 6
title: "Injection Cypher Neo4j"
description: "Le Cypher construit par chaîne contre les bases graphe Neo4j permet à un attaquant d'altérer les patterns MATCH, de franchir les frontières de label, d'abuser d'APOC et d'inférer des données en aveugle."
keywords:
  - Neo4j
  - injection Cypher
  - injection de base graphe
  - abus d'APOC
  - injection de graphe aveugle
---

# Neo4j

Neo4j est une base de données graphe interrogée avec **Cypher**, et Cypher est injectable pour la même raison que SQL : lorsque le code de l'application concatène l'entrée utilisateur dans une requête au lieu de lier des paramètres (`$param`), l'entrée est analysée comme syntaxe de requête. Un attaquant qui atteint un `MATCH` ou un `WHERE` construit par chaîne peut réécrire des patterns et des prédicats, s'échapper d'un littéral de chaîne et ajouter d'autres clauses avec `WITH`, `MATCH`, `CALL` et des commentaires (`//`, `/* */`).

Comme Cypher parcourt un unique graphe de propriétés sans frontières de tables, l'injection franchit librement les **labels** : un seul `MATCH (n) RETURN n` injecté peut énumérer chaque nœud, label, propriété et relation. Lorsque la bibliothèque de procédures `apoc` est installée, l'injection de `CALL` étend la portée au SSRF, à l'exfiltration sortante et à l'exécution de commandes. Lorsqu'aucune ligne n'est reflétée, l'inférence booléenne et time-based récupère les données en aveugle.

Ce sous-arbre couvre les mécaniques d'injection de base, l'extraction inter-labels, l'abus de procédures APOC et l'inférence aveugle.

## Pages

- **[Abus de procédures APOC](apoc-procedure-abuse.md)** : atteindre les procédures APOC via l'injection de CALL transforme une faille Cypher en SSRF, exfiltration sortante, accès aux fichiers et écritures de graphe arbitraires sur Neo4j.
- **[Injection aveugle](blind-injection.md)** : récupérer des données via une injection Cypher Neo4j qui ne renvoie aucune ligne, en utilisant des patterns conditionnels et apoc.util.sleep() pour l'inférence booléenne et time-based.
- **[Injection Cypher](cypher-injection.md)** : concaténer l'entrée utilisateur dans les clauses MATCH/WHERE permet à un attaquant d'échapper les littéraux de chaîne, d'inverser des prédicats et de chaîner des clauses Cypher contre Neo4j.
- **[Extraction de données](data-extraction.md)** : extraction inter-labels via des clauses injectées UNION, WITH et MATCH (n) RETURN n, plus énumération de schéma avec db.labels() et db.schema dans Neo4j.

## Outils

- **cypher-shell** : le shell Cypher natif pour tester les payloads contre une base de données.
- **Burp Suite** : intercepter et altérer les requêtes web adossées à Cypher.

## Références

- [Neo4j : Cypher manual](https://neo4j.com/docs/cypher-manual/current/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
