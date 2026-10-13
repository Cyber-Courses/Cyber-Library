---
title: "Injection Exp4j : abuser d'un évaluateur mathématique sandboxé"
description: "Exp4j ne parse que des nombres, des opérateurs et des fonctions enregistrées par l'hôte, donc une entrée non fiable dans une expression Exp4j atteint la grammaire mathématique, les fonctions et variables personnalisées exposées, et l'épuisement de ressources, pas le runtime de la JVM."
keywords:
  - Exp4j
  - injection Exp4j
  - évaluateur d'expression
  - abus de fonction personnalisée
  - déni de service
---

# Exp4j

Exp4j est un petit évaluateur d'expressions mathématiques pour la JVM. Il parse une chaîne en un arbre syntaxique abstrait de nombres, d'opérateurs, de parenthèses, d'un ensemble fixe de fonctions intégrées (`sin`, `cos`, `log`, `sqrt`, `abs`, etc.) et des variables et fonctions personnalisées que l'application hôte a enregistrées via l'`ExpressionBuilder`. Il évalue cet arbre en un unique `double`.

C'est tout le langage. Exp4j n'a aucune notion de types Java, pas de réflexion, pas de construction d'objets et pas d'accès au runtime. Une expression ne peut pas nommer une classe, ne peut pas appeler une méthode sur un objet, et ne peut pas atteindre `Runtime` ou `ProcessBuilder`. Injecter dans une expression Exp4j ne donne pas d'exécution de code, et un payload comme `Runtime.getRuntime().exec(...)` échoue simplement au parsing. Présenter Exp4j comme un sink de RCE est faux, et une évaluation qui le prétend ne survivra pas à la validation.

Ce que l'injection dans une expression Exp4j donne réellement, c'est le contrôle du calcul, borné par ce que l'hôte a exposé.

## Pattern vulnérable

```java
// formula taken from the request
Expression e = new ExpressionBuilder(formula)
    .variables("x", "y")
    .build()
    .setVariable("x", xValue)
    .setVariable("y", yValue);
double result = e.evaluate();
```

L'application attend que l'utilisateur fournisse quelque chose comme `x * 1.2 + y`. Parce que la chaîne brute est remise à l'`ExpressionBuilder`, l'utilisateur fournit à la place n'importe quelle expression que la grammaire accepte.

## Variables et fonctions personnalisées exposées

La surface atteignable est définie par le contexte que l'hôte a construit. Toute variable enregistrée avec `variables(...)` ou `setVariable(...)` est lisible, donc une formule injectée peut tirer des valeurs que l'interface n'a jamais voulu combiner ou renvoyer, par exemple en réfléchissant directement une variable sensible enregistrée comme résultat :

```
discount_rate
```

Les fonctions personnalisées enregistrées avec `.function(new Function("name", argc){...})` sont appelables par leur nom. Exp4j lui-même ne peut rien faire de dangereux, mais si l'application a enregistré une fonction personnalisée qui fait quelque chose de significatif (une recherche, une conversion de devise qui touche un backend, une fonction qui lit une valeur configurée), cette fonction est atteignable depuis l'expression injectée avec des arguments choisis par l'attaquant :

```
lookup(99999) + rate(payload)
```

C'est le seul cas où l'injection Exp4j dépasse l'arithmétique, et elle va exactement aussi loin que la fonction enregistrée, et pas plus loin. Énumérer quels noms de fonction et de variable se résolvent, par opposition à ceux qui lèvent une erreur de fonction ou de variable inconnue, cartographie le contexte disponible avant de façonner un payload.

## Épuisement de ressources

Parce que l'évaluation est de l'arithmétique pure sur `double`, l'abus qui ne dépend pas du contexte de l'hôte consiste à rendre cette arithmétique coûteuse ou dégénérée. Une grande exponentiation et des opérations profondément chaînées forcent un calcul lourd par requête :

```
9^9^9^9
```

Des parenthèses profondément imbriquées et de très longues chaînes d'opérateurs poussent le parsing et la construction de l'arbre, et des expressions qui mènent les résultats à `Infinity` ou `NaN` (division par zéro, `log(0)`, factorielles massives là où elles sont exposées) peuvent propager une valeur surprenante dans ce que l'application fait ensuite du `double` renvoyé en aval. Rien de tout cela n'échappe à l'évaluateur ; l'impact est la disponibilité et des valeurs calculées corrompues, pas l'exécution.

## Outils

- Tests manuels avec Burp Repeater ; payloads façonnés par moteur.
- **Burp Intruder** : énumérer les variables et fonctions personnalisées enregistrées par leur nom.

## Références

- [Exp4j documentation](https://www.objecthunter.net/exp4j/)
- [Exp4j custom functions and operators](https://www.objecthunter.net/exp4j/#Custom_functions)
