---
title: "Injection d'évaluation d'expression Go"
description: "Les applications Go embarquent des mini-langages d'expression sandboxés comme expr, où l'injection manipule les décisions de logique et de policy et où la seule voie vers un comportement dangereux est une méthode sur un objet exposé par l'hôte."
keywords:
  - injection d'expression Go
  - expr-lang
  - langage d'expression sandboxé
  - évaluation de policy
  - méthodes d'environnement
  - manipulation de logique
---

# Go

Les services Go embarquent un langage d'expression lorsqu'une règle, un filtre ou une policy doit être configurable au runtime : feature flags, règles de tarification, conditions d'alerte et policies d'accès sont courants. La bibliothèque dominante est `expr` (`github.com/expr-lang/expr`), un langage délibérément sandboxé et non Turing-complet. Contrairement à une EL JVM ou à un `eval` de scripting, il ne peut pas importer de packages ni appeler du Go arbitraire, donc il n'y a pas de primitive d'exécution de code inhérente.

Cette conception façonne tout le modèle d'attaque. Lorsque la chaîne d'expression est contrôlée par l'attaquant, le gain est de subvertir la décision que l'expression pilote, et la seule route vers quelque chose de plus lourd est d'appeler une méthode sur un objet que l'hôte a placé dans l'environnement d'évaluation. Si cet environnement expose un type dont les méthodes effectuent des I/O ou de l'exécution, ces méthodes sont atteignables ; sinon, le plafond est la manipulation de logique et de policy plus l'abus de ressources.

## Moteurs

- **[Expr](expr.md)** : le langage `expr-lang/expr`, son modèle d'environnement, et comment l'injection atteint la manipulation de logique et les méthodes exposées par l'hôte.

## Ce que l'environnement décide

Tout ce qui est atteignable dans une expression `expr` provient de l'`env` que l'hôte passe au moment de la compilation et de l'exécution. La grammaire peut lire ces variables, indexer ces maps et appeler les méthodes que ces objets exposent, et rien d'autre. L'évaluation de toute cible d'expression Go est donc une énumération de l'environnement : quelles variables, quels types, et lesquelles de leurs méthodes sont dans le scope. Cette frontière est toute la surface.

## Outils

- Tests manuels avec Burp Repeater ; payloads façonnés par moteur.
- **Burp Intruder** : énumérer les variables, types et méthodes exposés dans l'env.

## Références

- [expr-lang documentation](https://expr-lang.org/)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
