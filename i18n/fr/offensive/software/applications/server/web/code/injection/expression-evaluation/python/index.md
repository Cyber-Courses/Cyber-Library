---
title: "Injection par évaluation d'expression et de langage de politique Python"
description: "Puits d'évaluation Python côté serveur : eval et exec atteignent l'exécution de code arbitraire, tandis que Common Expression Language reste en sandbox et s'oriente vers l'abus de politique et de logique."
keywords:
  - injection eval Python
  - exécution de code exec
  - Common Expression Language
  - contournement de politique CEL
  - évaluation d'expression
---

# Python

Les applications Python évaluent des expressions atteignables par l'attaquant à deux endroits très différents, et les deux ne se comportent pas du tout pareil une fois l'entrée arrivée. Les `eval` et `exec` propres au langage compilent et exécutent du Python arbitraire, donc toute entrée qui les atteint est un chemin direct vers l'exécution de code sur l'hôte. Les langages de politique embarqués tels que Common Expression Language ne sont délibérément pas du Python : ce sont des évaluateurs en sandbox utilisés pour le contrôle d'admission, l'autorisation et la validation, donc le même état d'esprit d'injection produit une manipulation de logique et de politique plutôt qu'un shell.

Lisez les deux pages ensemble, car l'erreur pratique est de traiter l'une comme l'autre. Un puits d'expression qui exécute du vrai Python est noté comme exécution de code à distance ; une expression CEL qu'un opérateur croyait n'être « qu'un filtre » est notée comme un contournement de politique ou d'autorisation, pas comme une RCE.

- [eval() et exec()](python-eval.md) couvre l'exécution de code arbitraire véritable au travers de l'interpréteur, l'abus des builtins, le parcours de gadgets par traversée de classes qui survit aux builtins restreints, et pourquoi `ast.literal_eval` est le voisin sûr.
- [Common Expression Language](common-expression-language.md) couvre le cas en sandbox : manipuler les décisions d'autorisation et d'admission, divulguer le contexte exposé à l'expression, et abuser de toute fonction dangereuse enregistrée par l'hôte.

## Outils

- **Burp Suite** : Repeater et Intruder pour atteindre les puits eval/exec et CEL.
- **tplmap** : détecte et exploite les puits d'injection de code eval()/exec() Python.

## Références

- [Python: eval built-in](https://docs.python.org/3/library/functions.html#eval)
- [Python: ast.literal_eval](https://docs.python.org/3/library/ast.html#ast.literal_eval)
- [CEL specification](https://github.com/google/cel-spec)
