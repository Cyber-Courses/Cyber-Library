---
order: 5
title: "Injection Elasticsearch et OpenSearch"
description: "Lorsqu'une entrée utilisateur atteint la syntaxe Lucene query_string, le corps JSON Query DSL ou des scripts Painless, l'injection redevient possible sur les clusters Elasticsearch et OpenSearch."
keywords:
  - injection Elasticsearch
  - injection OpenSearch
  - Lucene query_string
  - Query DSL
  - scripting Painless
---

# Elasticsearch

Elasticsearch et son fork OpenSearch sont des stockages de documents à schéma flexible, interrogés via HTTP, et leur surface de requête est inhabituellement large. Trois couches distinctes acceptent des entrées non fiables et rouvrent l'injection. La syntaxe **Lucene `query_string` / `simple_query_string`** permet à un champ de recherche de transporter des opérateurs booléens, des sélecteurs de champ, des jokers, des plages et des tokens réservés, tous interprétés plutôt qu'échappés. Le corps de requête **JSON Query DSL** est fréquemment assemblé par concaténation de chaînes ou en étalant le JSON de la requête dans une clause `bool`, ce qui permet à un attaquant d'ajouter ou de remplacer des clauses. Et le **scripting Painless**, exposé via `script_fields`, les requêtes `script`, les tris et les mises à jour, exécute du code source influencé par l'attaquant lorsque l'entrée est interpolée dans le texte du script.

Comme les mêmes API sous-tendent les deux moteurs, les payloads présentés ici s'appliquent aussi bien à Elasticsearch qu'à OpenSearch. Les pages filles découpent la surface par sink : syntaxe query-string, corps Query DSL, injection de script Painless et extraction aveugle booléenne.

## Pages

- **[Extraction aveugle booléenne](blind-boolean.md)** : lorsque seul le résultat hit-ou-pas-hit est observable, des conditions de champ soigneusement conçues sur query_string, Query DSL ou les requêtes script permettent à un attaquant d'inférer et d'extraire les valeurs des documents...
- **[Injection Query DSL](query-dsl-injection.md)** : lorsque l'entrée utilisateur est concaténée dans le JSON Query DSL ou étalée dans une clause bool, un attaquant injecte des clauses should/must_not ou remplace la requête pour...
- **[Injection query string](query-string-injection.md)** : lorsqu'un terme de recherche circule non échappé dans query_string ou simple_query_string, les opérateurs Lucene permettent à un attaquant de recibler des champs, d'inverser la logique booléenne et d'élargir...
- **[Injection de script](script-injection.md)** : lorsqu'une entrée utilisateur est interpolée dans le source d'un script Painless dans script_fields, les requêtes script, les tris ou les mises à jour, un attaquant lit des données de document arbitraires...

## Outils

- **curl** : envoyer des requêtes _search et script brutes à l'API HTTP du cluster.
- **Burp Suite** : intercepter et altérer les requêtes web adossées à la recherche.

## Références

- [Elasticsearch : Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
