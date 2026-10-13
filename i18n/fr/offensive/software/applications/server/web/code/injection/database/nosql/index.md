---
order: 1
title: "Injection NoSQL"
description: "Injection contre des stockages de données non relationnels, où des objets, opérateurs ou fragments de langage de requête contrôlés par l'attaquant sont interprétés par le driver au lieu d'être traités comme des données."
keywords:
  - injection NoSQL
  - MongoDB
  - Cassandra
  - Redis
  - injection d'opérateur
  - injection Cypher
---

# NoSQL

L'injection NoSQL adapte l'idée d'injection aux stockages de données non relationnels, où la primitive dangereuse n'est pas toujours une chaîne de requête.

Elle couvre l'injection d'opérateur et de syntaxe dans les stockages de documents (MongoDB), l'injection de langage de requête dans les bases wide-column (Cassandra CQL) et graphe (Neo4j Cypher), l'injection de search-DSL (Elasticsearch), l'injection de commande et de scripting dans les stockages clé-valeur (Redis et les vues CouchDB), et l'injection d'expression dans les stockages managés (DynamoDB). Comme les techniques diffèrent nettement d'un moteur à l'autre, ce sous-arbre est organisé par base de données. Un fil commun traverse beaucoup d'entre elles : un objet ou un opérateur contrôlé par l'attaquant que le driver interprète au lieu de le traiter comme une donnée brute, souvent atteignable lorsque le JSON de la requête est passé directement dans une requête.

## Sous-thèmes

- **[Cassandra](cassandra/index.md)** : le CQL construit par chaîne contre Apache Cassandra permet à un attaquant de s'échapper des valeurs entre guillemets, d'ajouter des prédicats, de forcer des scans à l'échelle du cluster et de faire passer des écritures supplémentaires...
- **[CouchDB](couchdb/index.md)** : Apache CouchDB est une base de documents HTTP/JSON dont les design documents exécutent du JavaScript côté serveur, de sorte que l'injection cible ces fonctions ainsi que la requête, la vue...
- **[DynamoDB](dynamodb/index.md)** : Amazon DynamoDB est sans schéma et clé-valeur, mais les API qui l'interrogent (instructions PartiQL et expressions de filtre/condition) restent injectables lorsqu'elles sont construites...
- **[Elasticsearch](elasticsearch/index.md)** : lorsqu'une entrée utilisateur atteint la syntaxe Lucene query_string, le corps JSON Query DSL ou des scripts Painless, l'injection redevient possible sur les clusters Elasticsearch et OpenSearch.
- **[MongoDB](mongodb/index.md)** : injection d'opérateur et de syntaxe contre MongoDB, où le JSON de la requête atteint une requête sans assainissement et où des objets contrôlés par l'attaquant sont interprétés comme des opérateurs de requête...
- **[Neo4j](neo4j/index.md)** : le Cypher construit par chaîne contre les bases graphe Neo4j permet à un attaquant d'altérer les patterns MATCH, de franchir les frontières de label, d'abuser d'APOC et d'inférer des données en aveugle.
- **[Redis](redis/index.md)** : injection de commande via le protocole RESP et abus de commandes dangereuses dans un stockage clé-valeur Redis, du smuggling CRLF à la RCE par CONFIG SET et au Lua côté serveur.

## Outils

- **NoSQLMap** : détection et exploitation automatisées sur plusieurs moteurs NoSQL.
- **nosqli** : scanner d'injection NoSQL en Go pour les endpoints web.
- **Burp Suite** : intercepter et altérer les requêtes adossées à une base de données.

## Références

- [OWASP Web Security Testing Guide : Testing for NoSQL Injection](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
