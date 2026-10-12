---
title: "Dépose d'exécutables et de DLL : faire exécuter du code déposé depuis un partage"
order: 3
description: "Écrire des exécutables ou des DLL sur un partage mène à l'exécution de code lorsque quelque chose les lance : remplacer ou ajouter des programmes dans un partage de déploiement ou d'application, et la dépose de DLL où une application charge une bibliothèque par son nom depuis un chemin relatif au partage. La charge utile s'exécute dans le contexte de ce qui l'exécute, souvent un compte de service ou un administrateur."
keywords:
  - dll planting
  - dll hijacking
  - deployment share
  - executable planting
  - search order
---

# Dépose d'exécutables et de DLL

Écrire des fichiers de programme sur un partage ne compte que lorsque quelque chose les exécute, et les partages offrent plusieurs déclencheurs de ce type. Les partages de déploiement, de distribution de logiciels et d'applications contiennent des exécutables que des machines ou des utilisateurs lancent, de sorte qu'en remplacer ou en ajouter un fait exécuter du code attaquant sur ces machines. La dépose de DLL (hijacking) exploite la manière dont les applications localisent les bibliothèques : lorsqu'un programme charge une DLL par son nom et parcourt des répertoires incluant son propre dossier sur un partage, y placer une DLL malveillante de ce nom la fait charger dans le processus. La charge utile s'exécute avec l'identité de qui l'exécute, fréquemment un compte de service ou un administrateur déployant un logiciel.

## Voies

```bash
# 1. deployment / application share: add or replace an executable that gets run
smbclient //<t>/deploy -U 'user%pass' -c 'put evil.exe setup.exe'
# 2. DLL planting: identify a DLL an app loads from its (share-based) directory,
#    then plant a malicious DLL of that name next to the executable
#    - the app's import table / a procmon trace shows which DLLs it looks for
smbclient //<t>/apps -U 'user%pass' -c 'cd AppDir; put evil.dll version.dll'
```

Pour la dépose de DLL, la cible est une DLL que l'application charge par son nom et résout via un ordre de recherche qui inclut le propre répertoire de l'application (sur le partage). Les noms couramment détournables sont ceux que l'application s'attend à trouver localement ; une DLL malveillante exportant les fonctions requises (ou un proxy qui transmet vers la véritable) exécute son `DllMain` dans le processus.

## Notes d'exploitation

- La charge utile hérite de l'identité d'exécution : un partage de déploiement exécuté par des machines en tant que SYSTEM, ou une application lancée par un administrateur, fait de la dépose un gain de privilèges, pas seulement une exécution.
- La dépose de DLL nécessite que l'application charge une DLL par son nom avec un chemin de recherche qui atteint l'emplacement inscriptible du partage ; une DLL proxy/relais préserve la fonction de l'application pour qu'elle continue de fonctionner pendant que votre code s'exécute.
- Préférez un fichier que l'environnement exécute réellement selon une planification ou à l'ouverture de session/au déploiement, afin que l'exécution soit déterministe plutôt qu'opportuniste.
- Distinct de la coercition (qui capture des identifiants au simple survol), ceci exige que le fichier soit exécuté ; associez-le à la [coercition SCF et LNK](scf-and-lnk-coercion.md) lorsque vous n'obtenez qu'un utilisateur consultant le dossier.

## Références

- [MITRE ATT&CK : détournement de l'ordre de recherche des DLL](https://attack.mitre.org/techniques/T1574/001/)
- [MITRE ATT&CK : altération de contenu partagé](https://attack.mitre.org/techniques/T1080/)
