---
order: 1
title: "Injection FLWOR : réécrire l'itération et les prédicats XQuery"
description: "Une entrée non fiable à l'intérieur des clauses for/let/where/order by change quels nœuds sont itérés et quels prédicats sont vrais, permettant la rupture de littéral et les tautologies."
keywords:
  - injection FLWOR
  - injection XQuery
  - rupture de clause where
  - injection de prédicat XPath
  - or 1=1 XQuery
---

# Injection FLWOR

Une expression FLWOR (`for`, `let`, `where`, `order by`, `return`) est la construction d'itération de XQuery, l'analogue direct d'un `SELECT` SQL. Lorsque le code de l'application construit l'une de ces clauses en concaténant l'entrée de la requête, l'attaquant contrôle quelle séquence est liée, quel prédicat la filtre et comment les résultats sont ordonnés. Le motif vulnérable classique place l'entrée à l'intérieur d'une comparaison de chaînes :

```xquery
for $x in //user
where $x/name = '" + name + "'
return $x/profile
```

Avec `name` repris verbatim de la requête, le simple guillemet est la frontière à échapper.

## Rupture de littéral de chaîne

Fermer le guillemet et ajouter un prédicat transforme le filtre en tautologie. Fournir cette valeur pour `name` :

```
' or '1'='1
```

produit `where $x/name = '' or '1'='1'`, ce qui est vrai pour chaque `$x`, donc la boucle renvoie le profil de chaque utilisateur. La même chose fonctionne contre un prédicat construit en ligne :

```xquery
//user[name='" + name + "']
```

avec la charge utile `x'] | //user | a[''='` fermant le prédicat, unifiant l'ensemble complet `//user` avec `|`, et rouvrant un prédicat inoffensif pour que l'expression reste bien formée.

## Élargir la séquence itérée

Là où l'entrée se trouve dans le prédicat de liaison du `for`, l'attaquant élargit quels nœuds la boucle itère. En partant de :

```xquery
for $x in //user[role='" + role + "']
return $x/name
```

la valeur `' or '1'='1` rend le prédicat `role='' or '1'='1'`, donc la liaison sélectionne chaque `//user` quel que soit le rôle et la boucle renvoie tous leurs noms plutôt que ceux d'un seul rôle. Extraire des champs que la requête n'a jamais voulu renvoyer (hachages de mots de passe, jetons, drapeaux internes) nécessite une position d'expression plutôt qu'une liaison élargie : soit une requête dont la clause `return` est elle-même construite à partir de l'entrée, soit l'escalade vers des fonctions accessibles couverte dans l'abus de fonctions de bibliothèque. Lorsque les lignes ne sont pas du tout reflétées, l'inférence aveugle ci-dessous les récupère un caractère à la fois.

## Portée inter-document

Parce que les prédicats ne sont que des expressions booléennes, un prédicat injecté peut pivoter vers d'autres parties de l'arbre avec un chemin absolu. Le fragment `' or //secret/text() or '` force l'évaluation de `//secret` n'importe où dans le document, et la véracité de ce node-set fuite via le jeu de résultats ou via le timing.

## Inférence booléenne aveugle

Lorsque les lignes ne sont pas reflétées, un prédicat contrôlable fuite quand même un bit par requête. La charge utile `' or substring((//user[1]/password),1,1)='a` ne renvoie l'ensemble complet que lorsque le caractère deviné correspond, donc une recherche binaire ou linéaire scriptée sur `substring()` et `string-length()` reconstruit n'importe quelle valeur caractère par caractère. `fn:string-to-codepoints()` restreint chaque position avec une comparaison numérique au lieu d'un parcours d'alphabet.

## Outils

- **Burp Intruder** : itération de charges utiles FLWOR et de prédicat booléennes pour l'extraction aveugle.
- Tests manuels avec des charges utiles de rupture de tautologie `or` et `substring()`.

## Références

- [W3C XQuery 3.1: FLWOR Expressions](https://www.w3.org/TR/xquery-31/#id-flwor-expressions)
- [PayloadsAllTheThings: XPath Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
