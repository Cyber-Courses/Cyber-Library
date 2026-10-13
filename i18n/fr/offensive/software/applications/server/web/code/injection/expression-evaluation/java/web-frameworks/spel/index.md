---
title: "Injection SpEL : abus du Spring Expression Language"
description: "Spring évalue le SpEL issu des données de requête via SpelExpressionParser, et avec le contexte d'évaluation par défaut une référence de type atteint Runtime et ProcessBuilder pour l'exécution de code."
keywords:
  - injection SpEL
  - Spring Expression Language
  - SpelExpressionParser
  - "référence de type T()"
  - RCE SpEL
---

# SpEL

Le Spring Expression Language (SpEL) évalue des expressions face à un graphe d'objets à l'exécution. Il apparaît partout dans Spring : les annotations `@Value`, les règles d'accès Spring Security, le routage Spring Integration, les projections Spring Data, et tout code qui appelle directement un `SpelExpressionParser`. Lorsque des données de requête atteignent la chaîne analysée en une expression SpEL, via un paramètre réfléchi, une règle fournie par l'utilisateur ou une valeur de template, la grammaire SpEL s'exécute au sein de l'application.

SpEL est un langage objet complet. L'opérateur `T(...)` est une référence de type qui nomme n'importe quelle classe du classpath, et de là l'expression appelle des méthodes statiques, construit des objets et invoque des méthodes d'instance. Sous un `StandardEvaluationContext`, la valeur par défaut lorsque le code construit son propre parseur, cela atteint `java.lang.Runtime` sans restriction. Un `SimpleEvaluationContext` restreint la grammaire, donc la surface atteignable dépend du contexte fourni par l'application.

- **[Exécution de code](code-execution.md)** : les voies par référence de type et par réflexion d'une expression SpEL vers le runtime de l'hôte.
- **[Contournement de WAF](waf-bypass.md)** : réécrire le payload d'exécution de code pour défaire les filtres à signatures tout en restant un SpEL valide.

## Outils

- **Burp Suite** : Repeater et Intruder pour l'injection SpEL dans les puits Spring.
- **J2EEScan** : extension Burp avec des contrôles d'injection SpEL.

## Références

- [Spring Framework: Spring Expression Language](https://docs.spring.io/spring-framework/reference/core/expressions.html)
- [PayloadsAllTheThings: Java SpEL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
