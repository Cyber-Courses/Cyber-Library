---
title: "Injection par évaluation d'expression PHP"
description: "PHP expose des puits d'évaluation dynamique comme eval() qui exécutent du PHP arbitraire dans le processus interpréteur, transformant les chaînes d'expression contrôlées par l'attaquant en exécution de code à distance directe."
keywords:
  - injection d'expression PHP
  - eval
  - create_function
  - assert
  - évaluation de code dynamique
  - exécution de code à distance
---

# PHP

PHP se situe à l'extrémité opposée de l'éventail par rapport à un évaluateur en sandbox. Ses puits d'évaluation dynamique exécutent du PHP arbitraire dans le processus interpréteur, donc lorsqu'une entrée d'attaquant atteint l'un d'eux, le résultat est une exécution de code à distance directe et non restreinte. Il n'y a pas de grammaire à échapper ni de surface de types à énumérer : le PHP injecté s'exécute simplement, avec les privilèges et le jeu d'extensions du processus en cours.

Le puits principal est `eval()`, qui exécute une chaîne comme du PHP. La même classe d'exposition vient de `create_function` (qui construit un corps de fonction à partir d'une chaîne) et d'`assert` lorsqu'il reçoit un argument chaîne, qui tous deux évaluent leur contenu comme du code. Chacun prend une chaîne et l'exécute, c'est pourquoi ils appartiennent ici plutôt qu'au rendu de template.

## Moteurs

- **[eval()](eval.md)** : la construction de langage `eval()` et ses puits d'évaluation dynamique voisins, et le chemin d'un fragment injecté vers l'exécution de commande.

## Pourquoi l'impact est maximal

Comme la chaîne injectée est exécutée comme du PHP, l'attaquant hérite de tout le langage et du runtime : fonctions de système de fichiers, exécution de processus via `system`, `exec`, `shell_exec` et l'opérateur backtick, fonctions réseau, et quelles que soient les extensions et les identifiants détenus par le processus. Contrairement aux évaluateurs .NET et Go ailleurs dans cette section, il n'y a pas de plafond plus étroit à décrire. Les seules variables sont ce que le processus PHP peut faire sur l'hôte et la façon dont l'entrée atteint le puits.

## Outils

- **Burp Suite** : Repeater et Intruder pour atteindre les puits eval, assert et create_function.
- **tplmap** : détecte et exploite l'injection de code PHP basée sur eval().

## Références

- [PHP manual: eval](https://www.php.net/manual/en/function.eval.php)
- [OWASP: Code Injection](https://owasp.org/www-community/attacks/Code_Injection)
