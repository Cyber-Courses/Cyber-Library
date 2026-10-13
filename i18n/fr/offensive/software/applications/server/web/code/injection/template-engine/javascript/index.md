---
order: 3
title: "Injection de template côté serveur en JavaScript"
description: "SSTI dans les moteurs de template Node.js : les évasions prototype/constructor de Handlebars et le JavaScript inline de Pug pour atteindre require('child_process')."
keywords:
  - SSTI JavaScript
  - injection de template Node.js
  - Handlebars
  - Pug
---

# JavaScript

La SSTI Node.js atteint l'exécution de code via `require('child_process').execSync` (ou `process` / `global`), puisqu'un template capable d'exécuter du JavaScript peut charger n'importe quel module du cœur. Les moteurs diffèrent par la quantité de JavaScript qu'ils laissent le template exprimer.

Pug compile les templates en JavaScript et autorise du JS inline dans plusieurs constructions, de sorte que l'injection est proche d'une évaluation directe. Handlebars est pauvre en logique par conception et n'expose aucun JS direct, de sorte que son exploitation est un parcours constructor/prototype qui réassemble une fonction à l'exécution. La détection diffère aussi : Pug répond à `#{7*7}`, tandis que Handlebars utilise `{{ }}` mais n'évaluera pas `{{7*7}}` comme de l'arithmétique, de sorte qu'on l'identifie à ses messages d'erreur et à sa syntaxe de helpers.

## Moteurs

- **[Handlebars](handlebars.md)** : le parcours de prototype `constructor.constructor` / `#with` vers une fonction.
- **[Pug](pug.md)** : JavaScript inline dans l'interpolation et le code bufferisé vers `require('child_process')`.

## Références

- Documentation Handlebars et Pug
- PortSwigger Web Security Academy: Server-side template injection
