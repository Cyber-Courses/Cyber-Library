---
order: 3
title: "Moteurs d'expression Java : injection dans les bibliothèques d'évaluation embarquées"
description: "Évaluateurs JVM autonomes qu'une application embarque pour des formules, des règles et des templates, et comment le texte d'expression contrôlé par l'attaquant est atteint, séparés selon quels moteurs exposent le runtime et lesquels restent sandboxés."
keywords:
  - injection de moteur d'expression
  - injection JEXL
  - injection Seam EL
  - Exp4j
  - mXparser
---

# Moteurs d'expression

Les applications embarquent un évaluateur d'expressions autonome chaque fois qu'elles laissent les utilisateurs ou la configuration fournir une formule, un filtre, une règle ou un champ calculé plutôt qu'un calcul fixe. La bibliothèque parse la chaîne et l'évalue contre un contexte que l'hôte fournit. Lorsque cette chaîne porte une entrée non fiable, l'attaquant contrôle ce que l'évaluateur exécute, et le plafond dépend entièrement de ce que le moteur spécifique peut toucher.

Deux des moteurs présentés ici atteignent le runtime de la JVM et donnent l'exécution de code ; deux sont purement mathématiques et ne le font pas.

- [Exp4j](exp4j.md) : un évaluateur purement mathématique. Nombres, opérateurs et fonctions et variables enregistrées par l'hôte uniquement. Pas de réflexion, pas de types, pas de runtime, donc pas d'exécution de code inhérente.
- [JBoss Seam EL](jboss-seam-el.md) : Seam évalue l'EL unifié (`#{...}`), et une expression EL injectée atteint du Java arbitraire, donnant l'exécution de code.
- [JEXL](jexl.md) : les expressions Apache Commons JEXL peuvent construire des classes et invoquer des méthodes, atteignant `ProcessBuilder` et le runtime.
- [mXparser](mxparser.md) : un parseur mathématique scientifique avec des fonctions et constantes définies par l'utilisateur. Comme Exp4j, il n'a pas d'accès au runtime ni d'exécution de code inhérente.

## Outils

- **Burp Suite** : Repeater et Intruder pour sonder les sinks d'évaluateurs embarqués.
- **J2EEScan** : extension Burp avec des contrôles d'injection Java EL et d'expression.

## Références

- [Apache Commons JEXL](https://commons.apache.org/proper/commons-jexl/)
- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
