---
order: 3
title: "Évasion de sandbox et de restriction de fonctions : déjouer le confinement du runtime"
description: "S'échapper du confinement au niveau du runtime après exécution de code : contournements de disable_functions et open_basedir en PHP, évasions de sandbox Python, et évasions de vm Node.js."
keywords:
  - contournement de disable_functions
  - contournement de open_basedir
  - évasion de sandbox python
  - évasion de vm node
  - confinement du runtime
---

# Évasion de sandbox et de restriction de fonctions

Lorsqu'un attaquant a déjà l'exécution de code à l'intérieur d'un runtime mais que ce runtime est **confiné**, l'étape suivante est de s'échapper du confinement. Ces contrôles vivent à la couche runtime : `disable_functions` et `open_basedir` de PHP, les tentatives d'exécution restreinte de Python, et l'isolation du module `vm`/`vm2` de Node. Tous tentent de clôturer un langage qui n'a pas été conçu pour se clôturer lui-même, et chacun a ses évasions bien rodées.

## Quand cela s'applique

Vous atteignez cette section après un point d'appui : un webshell téléversé qui trouve `system` désactivé, une primitive SSTI ou eval confinée à un sandbox, un script utilisateur enveloppé dans `vm`. La question n'est plus « puis-je exécuter du code » mais « puis-je atteindre l'OS et le système de fichiers que le runtime tente de me cacher ».

## Les manœuvres récurrentes

- **Trouver un équivalent non bloqué.** Une denylist (`disable_functions`) couvre rarement tous les chemins vers la même capacité, donc énumérez ce qui reste.
- **Passer sous le langage.** Charger du code natif (LD_PRELOAD via un sous-processus lancé, FFI, une extension) pour que la restriction, appliquée dans l'interpréteur, ne s'applique plus.
- **Reconstruire la primitive.** Dans les sandboxes Python et Node, remontez le graphe d'objets jusqu'aux builtins, `process`, ou une classe qui fait le pont vers l'OS.

## Pages

- **[Contournement de disable_functions en PHP](php-disable-functions-bypass.md)** : atteindre l'exécution de commandes quand `disable_functions` bloque les appels évidents.
- **[Contournement de open_basedir en PHP](php-open-basedir-bypass.md)** : lire et écrire en dehors de l'arborescence de chemins autorisée.
- **[Évasion de sandbox Python](python-sandbox-escape.md)** : récupérer les builtins et l'accès OS depuis un `eval`/`exec` restreint.
- **[Évasion de vm Node.js](nodejs-vm-escape.md)** : s'échapper du module `vm`/`vm2` vers `process` et `require`.

## Références

- PHP manual: disable_functions, open_basedir
- HackTricks: bypass PHP restrictions (reference index)
