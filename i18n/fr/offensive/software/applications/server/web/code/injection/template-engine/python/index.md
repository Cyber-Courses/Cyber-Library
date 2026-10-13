---
order: 6
title: "Injection de template côté serveur en Python"
description: "SSTI dans les moteurs de template Python : les parcours de graphe d'objets de Jinja2 (Flask) vers la RCE et son sandbox, et l'évaluation Python directe de Mako."
keywords:
  - SSTI Python
  - Jinja2
  - Mako
  - injection de template Flask
  - évasion de sandbox
---

# Python

La SSTI Python est courante car Flask embarque Jinja2 et `render_template_string` reçoit fréquemment de l'entrée utilisateur. Les deux moteurs ici diffèrent fortement quant à la difficulté de la RCE.

Jinja2 restreint l'accès direct aux attributs dans son mode sandboxé mais expose un riche graphe d'objets, de sorte que l'exploitation consiste à cheminer depuis un objet exposé jusqu'au runtime Python. Mako ne pose aucune telle restriction : il évalue du Python directement à l'intérieur de `${ ... }`, donc la RCE est immédiate. La détection pour les deux commence avec `{{7*7}}` (Jinja2) ou `${7*7}` (Mako) renvoyant `49`.

## Moteurs

- **[Jinja2](jinja2.md)** : parcours de graphe d'objets (`cycler`, `config`, `lipsum`) vers `os`, et évasion de sandbox.
- **[Mako](mako.md)** : évaluation Python directe dans les blocs `${ }` et `<% %>`.

## Références

- Documentation Jinja2 et Mako (sandboxing, syntaxe d'expression)
- PortSwigger Web Security Academy: Server-side template injection
