---
title: "Évaluation d'expression JavaScript côté serveur et prototype pollution"
description: "Puits d'évaluation JavaScript côté serveur et prototype pollution : des clés non fiables empoisonnent Object.prototype, modifiant des valeurs par défaut partagées qui pilotent ensuite le contournement d'authentification et la RCE par chaîne de gadgets."
keywords:
  - injection JavaScript côté serveur
  - prototype pollution
  - Object.prototype
  - merge Node.js
  - chaîne de gadgets
---

# JavaScript

Le JavaScript côté serveur exécute des valeurs atteignables par l'attaquant dans Node.js, et deux classes de failles comptent ici. L'une est l'évaluation directe, où une entrée atteint une construction qui compile et exécute du JavaScript. L'autre est la prototype pollution, qui n'évalue jamais directement le code de l'attaquant mais corrompt l'`Object.prototype` partagé de sorte que les propriétés que l'application lit plus tard prennent les valeurs choisies par l'attaquant. La seconde est l'exposition côté serveur la plus subtile et la plus courante, parce que la requête de pollution et le point où le dommage se réalise sont généralement éloignés l'un de l'autre dans le code.

- [Prototype pollution](prototype-pollution.md) couvre le cas Node côté serveur : les puits de deep-merge et d'assignation récursive, les payloads de pollution `__proto__` et `constructor.prototype`, et l'escalade des valeurs par défaut empoisonnées vers le contournement d'autorisation et l'exécution de commande pilotée par gadgets.

La prototype pollution côté navigateur dans le DOM partage la cause racine mais a une surface de puits et un jeu de gadgets différents, et est traitée séparément.

## Outils

- **Burp Suite** : Repeater pour envoyer du JSON de pollution et sonder les gadgets en aval.
- **Server-Side Prototype Pollution Scanner** : extension Burp de PortSwigger détectant la pollution côté serveur.

## Références

- [OWASP: Prototype Pollution](https://cheatsheetseries.owasp.org/cheatsheets/Prototype_Pollution_Prevention_Cheat_Sheet.html)
- [PortSwigger: Prototype pollution](https://portswigger.net/web-security/prototype-pollution)
- [MDN: Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)
