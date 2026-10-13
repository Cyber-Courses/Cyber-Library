---
order: 2
title: "Injection d'opérateurs de tableau MongoDB avec des tableaux fournis par l'attaquant"
description: "Injecter $in, $nin, $all et $elemMatch à partir de tableaux contrôlés par l'attaquant élargit les correspondances, énumère des valeurs candidates et sonde les champs de type tableau."
keywords:
  - opérateurs de tableau
  - $in
  - $nin
  - $all
  - $elemMatch
  - injection MongoDB
---

# Opérateurs de tableau

Les opérateurs de tableau (`$in`, `$nin`, `$all`, `$elemMatch`) prennent un tableau comme argument, ce qui en fait le payload naturel lorsqu'un analyseur ou un corps JSON laisse un attaquant fournir un tableau là où l'application attendait un scalaire. Ils élargissent un ensemble de correspondances, énumèrent des valeurs candidates en une seule requête et atteignent les champs à valeur tableau.

La précondition est la même que d'habitude : la valeur atteint le filtre en tant que tableau ou objet contrôlé par l'attaquant, pas une chaîne convertie.

## $in et $nin pour élargir ou inverser une correspondance

`$in` matche un champ contre n'importe quelle valeur d'un tableau ; `$nin` matche tout document dont le champ n'est dans aucune d'elles. Fournir un ensemble large ou inversé transforme une recherche étroite en recherche large :

```json
{"role": {"$in": ["admin", "superuser", "root", "operator"]}}
{"status": {"$nin": ["banned"]}}
{"_id": {"$in": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]}}
```

`$nin` avec une courte liste d'exclusion renvoie presque tout, se faufilant au-delà d'un filtre censé sélectionner un seul enregistrement. Via un analyseur imbriqué, le tableau est construit avec des clés entre crochets répétées :

```
role[$in][]=admin&role[$in][]=root
_id[$nin][]=0
```

`$in` énumère aussi : fournissez un lot de valeurs candidates pour un champ protégé (noms de rôle, codes promo, feature flags) et une seule réponse vous indique si l'une d'elles matche, ce qui restreint l'ensemble pour la requête suivante.

## $all contre les champs à valeur tableau

`$all` exige qu'un champ (lui-même un tableau) contienne chaque élément que vous listez. Contre des champs comme `roles`, `permissions` ou `tags`, il teste l'appartenance avec précision :

```json
{"roles": {"$all": ["admin"]}}
{"permissions": {"$all": ["read", "write", "delete"]}}
```

Un tableau vide passé à `$all` ne matche aucun document, une branche fausse pratique lors de la construction d'un oracle ; un `$all` à un seul élément est un test d'égalité sur l'appartenance à un tableau qui confirme si une cible détient un rôle donné.

## $elemMatch pour interroger à l'intérieur des éléments d'un tableau

`$elemMatch` matche les documents où au moins un élément d'un champ tableau satisfait une sous-requête, et cette sous-requête peut elle-même porter des opérateurs injectés. Il atteint les documents imbriqués stockés dans des tableaux :

```json
{"accounts": {"$elemMatch": {"balance": {"$gt": 10000}}}}
{"sessions": {"$elemMatch": {"token": {"$regex": "^ey"}}}}
```

Comme l'objet interne accepte les opérateurs de comparaison et de regex, `$elemMatch` étend l'extraction aveugle (voir [Opérateurs de comparaison](comparison-operators.md) et [Regex](regex.md)) aux données contenues dans des champs de type tableau-de-sous-documents, une condition d'élément par requête. Combinez avec `$exists` pour d'abord confirmer qu'un champ au niveau de l'élément est présent (voir [Exists](exists.md)).

## Outils

- **Burp Repeater** : concevoir des tableaux $in, $nin et $all en JSON ou en clés entre crochets répétées.
- **nosqli** : scanner les paramètres à la recherche d'injection d'opérateur de tableau.
- **NoSQLMap** : automatiser l'énumération par payload de tableau.

## Références

- [MongoDB : $in](https://www.mongodb.com/docs/manual/reference/operator/query/in/)
- [MongoDB : $all](https://www.mongodb.com/docs/manual/reference/operator/query/all/)
- [MongoDB : $elemMatch](https://www.mongodb.com/docs/manual/reference/operator/query/elemMatch/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
