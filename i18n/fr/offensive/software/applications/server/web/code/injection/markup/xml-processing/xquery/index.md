---
order: 3
title: "Injection XQuery"
description: "Une expression XQuery construite par concaténation de chaînes contre des bases de données XML permet à un attaquant de réécrire les expressions de chemin et les prédicats, d'atteindre le système de fichiers et le réseau via des fonctions intégrées, et de lire des données à l'aveugle."
keywords:
  - injection XQuery
  - injection de base de données XML
  - BaseX
  - eXist-db
  - MarkLogic
---

# XQuery

XQuery est le langage de requête pour les bases de données XML telles que **BaseX**, **eXist-db** et **MarkLogic**, et il est injectable pour la même raison que SQL : lorsque le code de l'application concatène l'entrée utilisateur dans une requête au lieu de lier des variables externes, l'entrée est parsée comme de la syntaxe de requête. Un attaquant qui atteint un `for`, un `where` ou un prédicat construit par concaténation de chaînes peut s'échapper d'un littéral de chaîne, modifier la navigation XPath et ajouter d'autres expressions.

Parce qu'un processeur XQuery parcourt un document ou une collection sans frontières de colonnes, une seule expression injectée telle que `//*` peut énumérer chaque nœud dans la portée. Le langage embarque aussi une riche bibliothèque de fonctions : `doc()`, `fn:collection()` et `fn:unparsed-text()` atteignent fichiers et URL, et les extensions propriétaires (BaseX `proc:system`, `file:read-text`) atteignent le système de fichiers et le shell. Là où aucune donnée n'est reflétée, les erreurs dynamiques et statiques remontées au client soutiennent l'inférence aveugle.

Ce sous-arbre couvre la rupture de FLWOR et de prédicat, l'abus de fonctions de bibliothèque et d'extension, et l'extraction basée sur les erreurs.

## Pages

- **[Basé sur les erreurs](error-based-xquery.md)** : les erreurs statiques et dynamiques XQuery renvoyées au client fuitent du contenu, des types et de la structure via des conditions d'erreur contrôlées et des données intégrées dans les chaînes d'erreur...
- **[Injection FLWOR](flwor-injection.md)** : une entrée non fiable à l'intérieur des clauses for/let/where/order by change quels nœuds sont itérés et quels prédicats sont vrais, permettant la rupture de littéral et les tautologies.
- **[Abus de fonctions de bibliothèque](library-function-abuse.md)** : des fonctions intégrées et propriétaires accessibles comme doc(), fn:collection(), fn:unparsed-text() et BaseX proc:system transforment une injection en lecture de fichier, SSRF et RCE.

## Outils

- **Burp Suite** : fuzzing des points d'injection XQuery avec Intruder et Repeater.
- **Burp Collaborator** : confirmation du SSRF aveugle et de la portée hors bande depuis les fonctions intégrées.
- Tests manuels avec des charges utiles de rupture FLWOR et d'abus de fonctions.

## Références

- OWASP WSTG : Testing for XPath Injection
- OWASP WSTG : Testing for XML Injection
