---
title: "Injection de template côté serveur Handlebars"
description: "Exploiter la SSTI Handlebars dans Node.js : utiliser #with et le parcours constructor.constructor pour construire une Function qui appelle require('child_process') pour l'exécution de commande."
keywords:
  - SSTI Handlebars
  - constructor.constructor
  - helper with
  - require child_process
---

# Handlebars

Handlebars est délibérément pauvre en logique : il n'a pas de JavaScript inline et n'évaluera pas `{{7*7}}`, de sorte que la détection repose sur la sortie d'erreur distinctive du moteur (par exemple un `{{` malformé produisant une "Parse error" référençant Handlebars) et sur l'acceptation de la syntaxe des helpers `{{#if}}`/`{{#with}}`. Malgré les restrictions, Handlebars côté serveur est exploitable car le template peut atteindre le `constructor.constructor` d'un objet, c'est-à-dire le constructeur `Function` de JavaScript, et l'utiliser pour compiler du code arbitraire.

L'exploit utilise `#with` pour épingler une référence, chemine vers `constructor.constructor`, et l'appelle avec un corps de fonction qui require `child_process` :

```handlebars
{{#with "s" as |string|}}
  {{#with split as |conslist|}}
    {{this.pop}}
    {{this.push (lookup string.sub "constructor")}}
    {{this.pop}}
    {{#with string.split as |codelist|}}
      {{this.pop}}
      {{this.push "return process.mainModule.require('child_process').execSync('id').toString();"}}
      {{this.pop}}
      {{#each conslist}}
        {{#with (string.sub.apply 0 codelist)}}
          {{this}}
        {{/with}}
      {{/each}}
    {{/with}}
  {{/with}}
{{/with}}
```

La chaîne atteint `String.prototype.sub`, tire son `constructor` (le constructeur `Function`), et lui remet le corps à compiler et exécuter. Deux préconditions décident si cela se déclenche réellement.

D'abord, le détail de la portée de module : une fonction construite par le constructeur `Function` s'exécute dans la portée globale, où `require` n'est pas défini (en CommonJS, `require` est de portée module). Le corps atteint donc un chargeur via le global `process` à la place, `process.mainModule.require('child_process')`, plutôt qu'un simple `require(...)` qui lèverait une `ReferenceError`.

Ensuite, la restriction d'accès au prototype : depuis Handlebars 4.6.0 (2020), l'accès aux méthodes et propriétés de prototype est refusé par défaut, ce qui stoppe cette chaîne pendant qu'elle résout `split`, `sub`, `pop` ou `constructor`. Le payload n'est fiable que sur les versions plus anciennes, ou lorsque l'application réactive explicitement l'accès au prototype (`allowProtoMethodsByDefault` / `allowProtoPropertiesByDefault`, ou une allowlist personnalisée `allowedProtoMethods`). Sur un Handlebars actuel avec les options par défaut, il ne fonctionne pas du simple fait qu'une source contrôlée par l'attaquant atteint `Handlebars.compile`.

La précondition de base tient toujours : l'entrée non fiable doit être compilée comme template (`Handlebars.compile(userInput)`), et non passée comme donnée à un template fixe, lequel est échappé et sûr. Confirmez le sink de source de compilation et la version/les options de Handlebars avant d'investir dans le payload.

## Outils

- SSTImap

## Références

- Documentation Handlebars : helpers, compilation
- PortSwigger Web Security Academy: Server-side template injection
