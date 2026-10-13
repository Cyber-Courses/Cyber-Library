---
order: 1
title: "Injection query_string Elasticsearch via la syntaxe Lucene"
description: "Lorsqu'un terme de recherche circule non échappé dans query_string ou simple_query_string, les opérateurs Lucene permettent à un attaquant de recibler des champs, d'inverser la logique booléenne et d'élargir les jeux de résultats au-delà des filtres prévus."
keywords:
  - injection query_string
  - simple_query_string
  - syntaxe Lucene
  - ciblage de champ
  - _exists_
  - opérateurs booléens
---

# Injection query string

Les requêtes `query_string` et `simple_query_string` analysent un mini-langage (la syntaxe de requête Lucene) directement à partir du texte fourni. Lorsqu'une application insère le terme de recherche d'un utilisateur dans cette requête sans échapper les caractères réservés, le terme cesse d'être une donnée et devient de la **logique de requête**. Contrairement à SQL, il n'y a pas d'interpréteur hors-bande à subvertir : l'analyseur lui-même accorde la sélection de champ, la combinaison booléenne, les jokers, les plages et les tests d'existence.

## Schéma vulnérable

```json
POST /products/_search
{
  "query": {
    "query_string": { "query": "USER_TERM" }
  }
}
```

L'application substitue la valeur brute du champ de recherche dans `USER_TERM`. Les caractères réservés (`+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ /`) et les mots-clés `AND`, `OR`, `NOT`, `TO` sont tous actifs.

## Opérateurs booléens

Les recherches sur le champ par défaut peuvent être combinées ou niées. Si l'application attend un simple mot-clé, fournir des opérateurs réécrit l'intention :

```
laptop OR price:[0 TO 999999]
laptop AND NOT discontinued:true
```

`OR` avec une seconde clause large fait remonter des documents que le terme d'origine n'aurait jamais matchés.

## Ciblage de champ

`field:value` saute vers n'importe quel champ indexé, y compris ceux que l'interface n'expose jamais. Un attaquant qui connaît ou devine le mapping lit des champs hors du périmètre du champ de recherche :

```
name:laptop OR is_internal:true
* OR api_key:*
owner_id:12 OR owner_id:13
```

Un `*` en tête ou un terme vide combiné à un `OR` sur un champ sensible transforme une recherche de produit en dump des documents porteurs de ce champ.

## Jokers et plages

Les jokers (`*`, `?`) et les plages (`[min TO max]`, `{exclusif}`) élargissent les correspondances et permettent le sondage de valeurs :

```
email:*@corp.example
created:[2020-01-01 TO 2026-12-31]
price:<0 OR amount:>999999
```

Les jokers en tête (`*term`) sont rejetés par défaut mais souvent réactivés avec `allow_leading_wildcard` ; lorsqu'ils sont autorisés, `secret:*` confirme tout document qui possède le champ.

## Élargir avec `_exists_`

Le token `_exists_:field` matche tout document où un champ est présent, indépendamment de sa valeur, un moyen rapide de s'extraire d'une vue filtrée et d'énumérer quels documents portent des attributs privilégiés :

```
_exists_:password_reset_token
name:anything OR _exists_:ssn
```

## Contourner un filtre ajouté en suffixe

Les applications concatènent souvent un terme de confiance avec un filtre de périmètre en suffixe :

```json
{ "query": { "query_string": { "query": "USER_TERM AND tenant:acme" } } }
```

Un terme `x OR _exists_:id` produit `x OR _exists_:id AND tenant:acme`. Ne vous fiez pas à la priorité booléenne ici : l'analyseur de requête classique de Lucene n'applique pas la priorité standard `AND`-sur-`OR`, il compile vers une BooleanQuery plate de clauses MUST/SHOULD, et un `AND tenant:acme` en suffixe fait de `tenant:acme` une clause requise (MUST), de sorte que cette forme ne supprime pas de façon fiable le périmètre tenant. Le contournement fiable consiste à forcer le groupement là où le point d'injection autorise les parenthèses, en écartant purement et simplement la clause ajoutée :

```
x) OR (_exists_:id
```

Cela ferme prématurément le groupe prévu et en ouvre un sans périmètre, écartant entièrement le filtre ajouté.

## Indices d'analyseur et de boosting

Les opérateurs de flou (`term~`), de proximité (`"a b"~5`) et de boost (`term^10`) confirment que l'analyseur est actif lorsqu'une reconnaissance est nécessaire : une réponse qui traite `test~2` différemment de la chaîne littérale `test~2` prouve une analyse `query_string` plutôt qu'une requête `match`.

## Outils

- **curl** : lancer des recherches query_string avec des opérateurs Lucene injectés.
- **Burp Repeater** : livrer des payloads de ciblage de champ et de groupement via le paramètre de recherche.

## Références

- [Elasticsearch : Query string query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)
- [Elasticsearch : Simple query string query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-simple-query-string-query.html)
- [Apache Lucene : Query parser syntax](https://lucene.apache.org/core/documentation.html)
