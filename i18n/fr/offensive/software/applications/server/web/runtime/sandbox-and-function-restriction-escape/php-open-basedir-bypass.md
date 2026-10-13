---
title: "Contournement de open_basedir en PHP : lire et écrire en dehors de l'arborescence autorisée"
description: "S'échapper du confinement de chemin open_basedir de PHP pour accéder au système de fichiers plus large, via des particularités de traversée de répertoires, des symlinks, et des routes de sous-processus ou FFI que la vérification de l'interpréteur ne couvre pas."
keywords:
  - contournement de open_basedir
  - confinement de chemin
  - symlink
  - chdir
  - FFI
---

# Contournement de open_basedir en PHP

`open_basedir` confine les opérations de fichier PHP à une arborescence de chemins configurée. Comme `disable_functions`, il est appliqué à l'intérieur de l'API de fichier de l'interpréteur, de sorte que tout ce qui atteint le système de fichiers *en dehors* de cette API, ou qui trompe la vérification de chemin, y échappe. Cela compte lorsqu'un point d'appui doit lire une configuration/des secrets ou déposer un payload au-delà du répertoire autorisé.

## Astuces intra-interpréteur

Certains contournements restent dans PHP en abusant de la façon dont la vérification résout les chemins :

- **`chdir()` plus traversée relative** : entrer et sortir des répertoires autorisés par des séquences de `chdir()` et `..` a historiquement désynchronisé la base résolue de la base vérifiée sur certaines versions, autorisant un accès hors de l'arborescence.
- **Races de symlink (TOCTOU), et non simples symlinks** : PHP résout un symlink vers sa cible réelle *avant* d'appliquer `open_basedir`, donc un lien statique sous un répertoire autorisé qui pointe au-dehors est rejeté comme tout chemin extérieur. Le vecteur propre à la version est une race : permuter le chemin d'une cible légitime dans la base vers un symlink pointant au-dehors entre la vérification de PHP et son ouverture. Cela nécessite un bug concret de résolution ou de race sur le build PHP de la cible ; n'attendez pas qu'un simple symlink fonctionne.
- **`glob://` et particularités de wrappers** : les wrappers d'énumération listent parfois des entrées que la vérification directe refuserait.

Ceux-ci sont sensibles à la version ; testez contre le build PHP exact.

## Sortir de l'API de fichier

Les évasions robustes quittent entièrement la couche de fichier de l'interpréteur :

- **Sous-processus** : si un chemin d'exécution de commande est disponible (voir [contournement de disable_functions](php-disable-functions-bypass.md)), exécutez `cat`/`cp` comme processus enfant ; `open_basedir` ne s'applique pas au processus lancé.
- **FFI** : appelez les `fopen`/`open`/`read` de libc directement via `FFI::cdef`, contournant les fonctions de fichier vérifiées de PHP.
- **Code natif chargeable** : une bibliothèque `LD_PRELOAD` ou une extension chargée lit et écrit le vrai système de fichiers.

## Notes d'exploitation

- Vérifiez d'abord le confinement actuel et votre position :

  ```php
  echo ini_get('open_basedir');
  echo getcwd();
  ```

- `open_basedir` et `disable_functions` sont indépendants ; un hôte peut en fixer un et pas l'autre. Si l'exécution de commande est disponible, la route du sous-processus résout les deux d'un coup.
- Les astuces de traversée intra-interpréteur sont la seule option quand ni exécution ni FFI n'existent, gardez-les donc en réserve pour les points d'appui PHP-only étroitement verrouillés.

## Outils

- Snippets de contournement de `open_basedir` correspondant à la version ; one-liners FFI.

## Références

- PHP manual: open_basedir, FFI
- Public open_basedir bypass writeups (version-specific)
