---
title: "Désérialisation Node.js : node-serialize et bibliothèques de réanimation de fonctions"
description: "Exploiter la désérialisation JavaScript : la réanimation de fonction de node-serialize comme primitive de RCE immédiate via une IIFE, plus les sinks funcster et serialize-javascript."
keywords:
  - node-serialize
  - désérialisation javascript
  - IIFE
  - unserialize
  - funcster
---

# Désérialisation Node.js

JavaScript n'a pas de format de sérialisation d'objet intégré comme PHP ou Java, donc les bugs de désérialisation Node vivent dans des bibliothèques tierces qui sérialisent et réaniment des **fonctions**. Le fer de lance est `node-serialize` : son `unserialize()` reconstruit une fonction sérialisée et, via une expression de fonction immédiatement invoquée, l'exécute au moment du chargement, ce qui fait du sink une primitive de RCE directe.

## node-serialize

`node-serialize` encode les fonctions avec un marqueur `_$$ND_FUNC$$_`. À `unserialize()`, une valeur étiquetée comme fonction passe par `eval`. Ajouter `()` au corps de la fonction la transforme en IIFE qui s'exécute pendant la désérialisation :

```json
{"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('id',function(e,o){console.log(o)});}()"}
```

Le `()` final avant le guillemet fermant est le déclencheur : la fonction réanimée s'invoque elle-même immédiatement. Tout ce qui appelle `serialize.unserialize(userInput)` là-dessus exécute la commande. `require('child_process')` est accessible parce que l'eval s'exécute ici dans le scope du module (contrairement à un constructeur `Function` nu, qui est en scope global).

Livrez le JSON là où l'application le désérialise : cookies, corps, entrées de cache ou messages de file.

## Autres bibliothèques

- **`funcster`** : reconstruit les fonctions dans un module en sandbox ; des motifs d'évasion atteignent `this.constructor.constructor('return process')()` pour regagner `process` et `require`.
- **`serialize-javascript`** : avant tout un sérialiseur, mais le fait de faire transiter sa sortie par une réanimation basée sur `eval` dans le code applicatif réintroduit le sink d'exécution de fonction.
- **`cryo`, `node-cryo`** : sérialiseurs de graphe d'objets qui peuvent réanimer des fonctions.

## Notes d'exploitation

- Confirmez la bibliothèque et que votre entrée atteint son appel `unserialize`/de réanimation ; un simple `JSON.parse` n'est pas vulnérable (il ne réanime jamais de fonctions).
- Pour la sortie, le callback de `child_process.exec` ne vous revient pas par HTTP ; préférez un callback OOB (DNS/HTTP) ou un reverse shell.
- Si la réanimation utilise le constructeur `Function` (scope global) plutôt qu'`eval` (scope module), atteignez un loader via `process.mainModule.require('child_process')` au lieu d'un `require` nu.

## Outils

- Payloads `_$$ND_FUNC$$_` forgés à la main.

## Références

- PortSwigger Web Security Academy: Insecure deserialization
- node-serialize advisory writeups
