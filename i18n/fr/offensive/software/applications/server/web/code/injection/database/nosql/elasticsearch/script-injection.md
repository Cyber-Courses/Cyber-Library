---
order: 3
title: "Injection de script Painless Elasticsearch"
description: "Lorsqu'une entrée utilisateur est interpolée dans le source d'un script Painless dans script_fields, les requêtes script, les tris ou les mises à jour, un attaquant lit des données de document arbitraires et exécute du code à l'intérieur du bac à sable de scripting."
keywords:
  - injection Painless
  - script_fields
  - requête script
  - injection de script
  - scripting Elasticsearch
  - évasion de bac à sable
---

# Injection de script

Painless est le langage de scripting par défaut d'Elasticsearch, accessible via `script_fields`, les requêtes `script`, le tri basé sur script, `function_score` et `_update` / `_update_by_query`. Lorsqu'une application construit un script en **interpolant l'entrée utilisateur dans la chaîne source du script** plutôt qu'en la passant comme paramètre lié, l'entrée devient du Painless exécutable, offrant à un attaquant une divulgation de données calculée et une exécution de code au sein du bac à sable de scripting.

Le scripting doit être activé pour ces sinks. Le Painless inline et stocké est activé par défaut dans de nombreux déploiements, bien que `script.allowed_types` / `script.allowed_contexts` puissent les restreindre.

## Schéma vulnérable

```js
const body = {
  query: { match_all: {} },
  script_fields: {
    calc: { script: { source: `doc['price'].value * ${userInput}` } }
  }
};
```

Le `userInput` attendu est un multiplicateur, mais il est concaténé dans le source. Une valeur de `0; return doc['ssn'].value` (ou toute expression Painless valide) est analysée et exécutée.

## Divulgation de données via les champs calculés

L'abus le plus simple renvoie des champs que la requête n'était jamais censée exposer, calculés par hit :

```json
{
  "query": { "match_all": {} },
  "script_fields": {
    "leak": { "script": { "source": "doc['password_hash'].value" } }
  }
}
```

Un code source qui référence `doc['field']` lit tout champ indexé avec `doc_values`, contournant le filtrage `_source` sur lequel l'application s'appuie.

## Les conditions scriptées comme oracle

Une requête `script` évalue un booléen par document. Le source injecté transforme la recherche en primitive d'inférence :

```json
{
  "query": {
    "script": {
      "script": {
        "source": "doc['role'].value == 'admin' && doc['secret'].value.length() > 40"
      }
    }
  }
}
```

Les différences de nombre de hits laissent fuiter des attributs de documents sans les renvoyer directement.

## Atteindre le bac à sable

Painless est mis en bac à sable, mais le source injecté peut tout de même exercer la surface d'API autorisée : manipulation de chaînes et de collections, appels à réflexion limitée et regex. Un regex catastrophique ou un source à grande boucle provoque un épuisement de calcul :

```painless
int x = 0; for (int i = 0; i < 100000000; i++) { x += i; } return x;
```

Lorsque le déploiement expose des contextes supplémentaires ou une build plus ancienne avec une liste d'autorisation plus faible, le même point d'interpolation est le pied-à-terre pour des chaînes d'évasion de bac à sable qui atteignent `Runtime` ou les API de système de fichiers ; le mécanisme d'injection (source non lié) est identique quelle que soit la distance à laquelle le bac à sable peut être poussé.

## Injection via les mises à jour

`_update_by_query` exécute du Painless avec accès en écriture à `ctx._source`, de sorte qu'un script de mise à jour interpolé mute les documents :

```json
POST /users/_update_by_query
{
  "query": { "term": { "name": "victim" } },
  "script": { "source": "ctx._source.role = 'admin'" }
}
```

Si l'application construit le `source` à partir de l'entrée (par exemple une fonctionnalité « mettre le champ X à Y »), l'attaquant écrase des champs arbitraires sur des documents arbitraires.

## L'indice de paramétrage

Le code sûr passe les valeurs par `params` (`"source": "doc['price'].value * params.m", "params": { "m": userInput }`) afin que l'entrée n'atteigne jamais le source. Offensivement, l'indicateur est tout `script.source` contenant une concaténation de chaînes, des template literals ou des placeholders de format alimentés par des données de requête.

## Outils

- **curl** : soumettre des corps script_fields et _update_by_query avec du source Painless injecté.
- **Burp Repeater** : concevoir des requêtes qui interpolent des payloads Painless dans le sink.

## Références

- [Elasticsearch : How to use scripts](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html)
- [Elasticsearch : Painless scripting language](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-guide.html)
- [Elasticsearch : Script fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#script-fields)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
