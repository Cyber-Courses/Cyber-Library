---
title: "Injection mXparser : abuser d'un parseur mathématique scientifique"
description: "mXparser évalue des expressions mathématiques avec des fonctions, arguments et constantes définis par l'utilisateur mais n'a aucun accès aux types Java ni au runtime, donc l'injection atteint la surface mathématique, les éléments définis par l'utilisateur exposés et l'abus de ressources plutôt que l'exécution de code."
keywords:
  - mXparser
  - injection mXparser
  - parseur d'expression mathématique
  - fonction définie par l'utilisateur
  - déni de service
---

# mXparser

mXparser est un parseur d'expressions mathématiques riche en fonctionnalités pour la JVM et .NET. Il gère une large bibliothèque de fonctions intégrées (trigonométriques, hyperboliques, spéciales, de probabilité, combinatoires), des opérateurs, des opérateurs itérés et de sommation, des constantes physiques et mathématiques, et des éléments définis par l'utilisateur : les objets `Argument`, `Constant` et `Function` que l'hôte enregistre sur l'`Expression` avant le parsing.

Malgré cette ampleur, mXparser reste un moteur de mathématiques. Il évalue vers un résultat numérique et n'a aucun mécanisme pour référencer des classes Java, construire des objets, invoquer des méthodes, ou atteindre la réflexion, `Runtime` ou `ProcessBuilder`. Une expression mXparser injectée ne produit donc pas d'exécution de code, et les payloads de commandes shell ne sont pas une syntaxe valide dans la grammaire. Décrivez l'impact honnêtement : le vecteur est la surface mathématique et les éléments définis par l'hôte, pas la JVM.

## Pattern vulnérable

```java
// formula supplied in the request
Expression ex = new Expression(formula);
ex.addArguments(new Argument("x", xValue));
double result = ex.calculate();
```

L'entrée prévue est une formule arithmétique en `x`. La chaîne brute atteint le parseur, donc un attaquant fournit n'importe quelle expression que mXparser accepte.

## Éléments définis par l'utilisateur exposés

Le contexte atteignable est tout ce que l'application a enregistré. Les `Argument`s et `Constant`s définis par l'utilisateur sont lisibles par leur nom, donc une expression injectée peut faire remonter une valeur que l'interface n'avait jamais voulu exposer en la nommant directement. Les `Function`s définies par l'utilisateur ajoutées avec `addFunctions(...)` sont appelables avec des arguments choisis par l'attaquant :

```
pricing(0) + secretMargin
```

Comme pour tout moteur de math, mXparser ne fait rien de dangereux par lui-même ; la fonction enregistrée va exactement aussi loin que l'hôte l'a implémentée. Sonder quels identifiants s'évaluent par opposition à ceux qui laissent l'expression dans un état d'erreur (mXparser renvoie `NaN` et expose `getErrorMessage()`) énumère les arguments, constantes et fonctions disponibles.

mXparser parse aussi des expressions qui construisent ses propres éléments définis par l'utilisateur inline, y compris la syntaxe de définition de fonction et d'itération, ce qui élargit la grammaire qu'une chaîne injectée peut exercer mais la maintient à l'intérieur du modèle numérique.

## Épuisement de ressources

L'abus indépendant du contexte consiste à forcer une évaluation coûteuse. Les opérateurs itérés sur de grandes plages font un travail substantiel par requête, par exemple une sommation sur un énorme intervalle d'indices :

```
sum(i, 1, 100000000, i^2)
```

Des fonctions profondément imbriquées, de grandes factorielles et évaluations gamma, et des expressions conçues pour renvoyer `NaN` ou des valeurs infinies poussent le coût de calcul et peuvent propager des nombres dégénérés dans la logique applicative en aval. L'impact est la disponibilité et des résultats corrompus, contenu dans l'évaluateur.

## Outils

- Tests manuels avec Burp Repeater ; payloads façonnés par moteur.
- **Burp Intruder** : énumérer les arguments, constantes et fonctions définis par l'utilisateur par leur nom.

## Références

- [mXparser documentation](https://mathparser.org/)
- [mXparser user-defined functions and arguments](https://mathparser.org/mxparser-tutorial/)
