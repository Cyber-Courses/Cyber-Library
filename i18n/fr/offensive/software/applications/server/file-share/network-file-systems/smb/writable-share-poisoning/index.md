---
title: "Empoisonnement de partage inscriptible : dépose de charges utiles et de fichiers de coercition d'identifiants"
order: 4
description: "Un partage SMB inscriptible est une primitive d'exécution et de capture d'identifiants. Un attaquant dépose des exécutables ou des DLL là où ils seront exécutés, empoisonne les documents et modèles Office que les utilisateurs ouvrent, et dépose des fichiers SCF et LNK dont les icônes forcent le visualiseur à s'authentifier auprès d'un hôte attaquant. L'accès en écriture transforme un serveur de fichiers en exécution de code et en identifiants capturés."
keywords:
  - writable share
  - dll planting
  - scf lnk
  - template poisoning
  - credential coercion
---

# Empoisonnement de partage inscriptible

L'accès en écriture à un partage est bien plus que la capacité d'y stocker des fichiers : c'est un moyen de faire exécuter du code et de capturer des identifiants quand d'autres utilisateurs ou systèmes interagissent avec le partage. Trois techniques suivent. Déposer des exécutables ou des DLL là où une application ou un utilisateur les exécutera donne l'exécution de code dans ce contexte. Empoisonner les documents Office et leurs modèles exécute des macros ou des références externes lorsqu'un utilisateur les ouvre. Et déposer des fichiers SCF ou LNK dont l'icône ou la cible pointe vers un hôte attaquant force la machine du visualiseur à s'y authentifier dès que le dossier est consulté, capturant ou relayant l'identifiant. Le fil conducteur est que l'accès en écriture convertit l'usage normal du partage en avantage pour l'attaquant.

```bash
# confirm writable, then plant (null/guest/creds per the enumeration)
smbclient //<target>/<share> -U 'user%pass' -c 'put payload'
nxc smb <target> -u user -p 'pass' --shares     # WRITE flag confirms the target
```

## Sous-thèmes

- **[Dépose d'exécutables et de DLL](executable-and-dll-planting.md)** : faire exécuter du code déposé.
- **[Empoisonnement d'Office et de modèles](office-and-template-poisoning.md)** : exécution de macros et de modèles à l'ouverture.
- **[Coercition SCF et LNK](scf-and-lnk-coercion.md)** : forcer l'authentification lors de la consultation d'un dossier.

## Références

- [MITRE ATT&CK : altération de contenu partagé](https://attack.mitre.org/techniques/T1080/)
- [NetExec : SMB](https://www.netexec.wiki/smb-protocol)
