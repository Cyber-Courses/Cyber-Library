---
title: "Machines virtuelles"
order: 1
description: "Prendre le contrôle des VM Azure : run command et Custom Script Extension pour une exécution SYSTEM ou root, vol du jeton d'identité managée, et secrets dans user data et cloud-init."
keywords:
  - machines virtuelles
  - run command
  - custom script extension
  - identité managée
  - user data
---

# Machines virtuelles

Une VM sur laquelle vous détenez des droits d'écriture est à la fois une cible d'exécution de code et une source d'identifiants. Le plan de gestion vous donne une exécution sans aucun identifiant stocké (run command, une extension), et une fois sur la machine, l'**identité managée** attachée à la VM distribue un jeton d'abonnement depuis le point de terminaison de métadonnées.

## Pages

- **[Run command](run-command.md)** : `runCommand` pour une exécution SYSTEM ou root sans identifiant.
- **[Custom Script Extension](custom-script-extension.md)** : `extensions/write` pour exécuter du code, y compris VMAccess pour réinitialiser le mot de passe admin.
- **[Identité managée](managed-identity.md)** : voler le jeton de l'identité de la VM depuis IMDS.
- **[User data](user-data.md)** : lire cloud-init et user data pour y trouver des secrets, ou les écrire pour s'exécuter au démarrage.

## Références

- [HackTricks Cloud : VM Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : escalade de privilèges Azure via VM run command](https://www.netspi.com/blog/technical-blog/)
- [Microsoft : run command](https://learn.microsoft.com/azure/virtual-machines/run-command-overview)
