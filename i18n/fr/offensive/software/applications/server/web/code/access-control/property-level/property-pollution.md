---
title: "Pollution de propriété : prototype pollution et abus de fusion profonde dans les stacks JavaScript"
description: "Injecter __proto__ ou constructor.prototype via des helpers récursifs de fusion, de clonage ou d'assignation par chemin afin que des propriétés choisies par l'attaquant apparaissent sur des objets auxquels le serveur fait confiance, contournant l'autorisation au niveau des propriétés."
keywords:
  - prototype pollution
  - __proto__
  - fusion profonde
  - BOPLA
---

# Pollution de propriété

## Contexte

En JavaScript et Node, chaque objet hérite de `Object.prototype`. Lorsque le code serveur fusionne un corps JSON contrôlé par l'attaquant dans un objet à l'aide d'un helper récursif de fusion profonde, de clonage ou d'assignation par chemin qui ne protège pas les clés `__proto__`, `constructor` et `prototype`, l'attaquant écrit sur ce prototype partagé. Ensuite, chaque objet du realm semble porter la propriété injectée, de sorte qu'un contrôle d'autorisation qui lit un champ tel que `user.isAdmin` ou `options.role` sur un objet fraîchement construit voit la valeur de l'attaquant alors qu'elle n'a jamais été définie sur cet objet. C'est un contournement d'autorisation au niveau des propriétés : vous définissez une propriété que vous n'avez jamais été autorisé à définir, sur des objets que vous ne possédez pas.

## Théorie

Le point d'injection (sink) est toute routine qui parcourt des clés choisies par l'attaquant vers une cible : fusion profonde codée à la main, `Object.assign` récursif, lodash `merge` / `set` / `defaultsDeep` (historiquement vulnérables), jQuery `$.extend(true, ...)` et les analyseurs de chaînes de requête qui construisent des objets imbriqués à partir de `a[b][c]=d`. Le payload utilise une clé que le parcoureur traite comme un nom de propriété ordinaire mais que le moteur traite comme le pointeur de prototype. Le gadget d'impact est toute propriété que l'application lit sur un objet qu'elle vient de construire sans l'avoir explicitement définie : indicateurs de rôle et d'habilitation, bascules de fonctionnalités, interrupteurs de validation, ou options de template et de configuration (qui peuvent pousser la prototype pollution plus loin vers du RCE ou du XSS).

## Pratique

### Polluer via un corps JSON fusionné

- Là où un endpoint fusionne le corps de la requête dans un objet utilisateur ou d'options, envoyez un corps portant `__proto__` :

```json
{ "__proto__": { "isAdmin": true } }
```

Si une réponse, une action ou une décision d'autorisation ultérieure traite alors le compte comme privilégié, le sink de fusion profonde a pollué le prototype. Lorsque la clé littérale `__proto__` est retirée, atteignez le même prototype via le constructeur :

```json
{ "constructor": { "prototype": { "isAdmin": true } } }
```

### Polluer via des paramètres de requête imbriqués

- Les analyseurs qui construisent des objets imbriqués à partir de la notation entre crochets acceptent la clé dans l'URL ou le corps du formulaire :

```
POST /api/profile?__proto__[role]=admin
```

Confirmez en relisant un objet que le serveur construit après la requête et en vérifiant la présence de la propriété injectée, ou en observant directement le changement de privilège.

Ceci est distinct de la pollution de paramètres HTTP (où le paramètre dupliqué gagne), traitée sous [pollution de paramètres](../../injection/http/parameter-pollution.md) ; ici le payload mute le prototype de l'objet plutôt que d'exploiter la précédence des clés dupliquées.

## Outils

- **Burp Suite**
- **curl**
- Scanners de gadgets de prototype pollution (serveur et client)
