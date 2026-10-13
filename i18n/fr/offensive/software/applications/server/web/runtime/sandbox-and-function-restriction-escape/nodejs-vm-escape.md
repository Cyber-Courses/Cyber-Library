---
title: "Évasion de vm Node.js : s'échapper de vm et vm2 vers process et require"
description: "S'échapper de l'isolation du module vm/vm2 de Node pour revenir au realm hôte : évasions par chaîne de constructeurs vers process.mainModule.require, et motifs d'évasion de sandbox vm2 connus."
keywords:
  - évasion de vm node
  - vm2
  - évasion de sandbox
  - constructor.constructor
  - process.mainModule
---

# Évasion de vm Node.js

Le module `vm` intégré de Node exécute du code dans un contexte séparé mais **ne fournit explicitement pas de frontière de sécurité** ; `vm2` était la bibliothèque communautaire qui tentait d'en être une. À partir d'une primitive de code confinée à un contexte `vm`/`vm2` (un système de plugins, un évaluateur de formules/d'expressions, une SSTI qui atterrit dans un sandbox), l'objectif est de remonter d'un objet du sandbox vers le `process` du realm hôte, et de là vers `require('child_process')`.

## La manœuvre centrale : la chaîne de constructeurs

Les objets passés dans le sandbox portent des prototypes qui remontent vers le constructeur `Function` de l'hôte, lequel compile du code dans le realm hôte :

```js
// CommonJS host (process.mainModule is defined):
this.constructor.constructor('return process')().mainModule.require('child_process').execSync('id').toString()
```

`constructor.constructor` est le constructeur `Function` ; `Function('return process')()` renvoie l'objet `process` de l'hôte. Les objets Error, les valeurs lancées (thrown) et les méthodes de prototype sont tous des points de départ utilisables lorsque l'accès direct aux objets est limité.

L'étape de loader dépend de la façon dont l'hôte a été démarré. `process.mainModule.require` n'existe que lorsque le point d'entrée est un module **CommonJS** ; si l'hôte s'exécute comme un module ES (un point d'entrée `.mjs`, ou `"type":"module"`), `process.mainModule` vaut `undefined` et ce payload échoue après avoir obtenu `process`. Utilisez un loader qui n'en dépend pas :

```js
// ESM / current Node (Node 22.3+): built-in module accessor
this.constructor.constructor('return process')().getBuiltinModule('child_process').execSync('id').toString()

// Any modern Node: dynamic import returns a promise
this.constructor.constructor('return process')().binding // (legacy, often restricted)
// or, awaited: (async()=>{return (await import('node:child_process')).execSync('id').toString()})()
```

`process.getBuiltinModule('child_process')` est le remplacement le plus propre et adapté à la version ; repliez-vous sur un `import('node:child_process')` dynamique là où il est indisponible.

## Spécificités de vm2

Le `vm` nu est trivial à échapper (ce n'a jamais été une frontière). `vm2` proxyait et gelait activement les objets hôtes, de sorte que les évasions ciblaient des failles dans ce proxying : atteindre un objet hôte non proxyé via une pile d'erreur, un trap `Proxy`, un accès basé sur `Symbol`, ou une fonction hôte fuitée dans le sandbox, puis l'utiliser pour obtenir le vrai `Function`/`process`. `vm2` a été retiré par ses mainteneurs au profit d'`isolated-vm` précisément parce que ces évasions ne cessaient de réapparaître, donc une cible exécutant encore `vm2` est un candidat sérieux ; faites correspondre une évasion publiée à sa version.

## Notes d'exploitation

- Confirmez quelle isolation est utilisée : `vm` nu (pas de frontière, la chaîne de constructeurs fonctionne directement), `vm2` (nécessite une évasion de proxy adaptée à la version), ou `isolated-vm` (une vraie frontière d'isolat V8, hors de portée de ces astuces, pivotez ailleurs).
- Choisissez le loader selon le système de modules de l'hôte : `process.mainModule.require` pour CommonJS, `process.getBuiltinModule(...)` ou `import()` dynamique pour ESM/Node actuel. Un `require` nu est indéfini dans le scope global de l'hôte où s'exécute un constructeur `Function`.
- Pour la sortie, renvoyez le résultat de la commande via l'expression évaluée (`.toString()`), ou utilisez un callback OOB lorsque le sink n'affiche rien.

## Outils

- Payloads de chaîne de constructeurs construits à la main ; PoC d'évasion vm2 publiés (correspondant à la version).

## Références

- Node.js docs: vm module (security caveat)
- vm2 advisories / isolated-vm project
