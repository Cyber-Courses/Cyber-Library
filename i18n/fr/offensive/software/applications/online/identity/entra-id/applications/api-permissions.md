---
title: "Permissions d'API : abus des app-roles Microsoft Graph"
order: 3
description: "Abuser des permissions d'application Microsoft Graph : tirer parti d'AppRoleAssignment et de portées à forte valeur telles que RoleManagement.ReadWrite.Directory pour s'élever jusqu'au contrôle du tenant."
keywords:
  - permissions d'API
  - Microsoft Graph
  - app roles
  - AppRoleAssignment
  - RoleManagement
---

# Permissions d'API

Les **permissions d'application** Microsoft Graph (app roles) sont accordées aux service principals et s'exécutent sans utilisateur connecté. Une poignée d'entre elles mettent fin au tenant : `RoleManagement.ReadWrite.Directory` permet à une application d'attribuer des rôles d'annuaire (y compris Global Admin), et `AppRoleAssignment.ReadWrite.All` permet à une application de s'accorder à elle-même n'importe quelle autre permission.

## S'élever via un app role privilégié

```bash
# find SPs holding dangerous app roles
az ad sp list --all --query "[?appRoles]" 
# with AppRoleAssignment.ReadWrite.All on an SP you control, grant it RoleManagement.ReadWrite.Directory
# POST /servicePrincipals/{id}/appRoleAssignments  (Graph)
# then assign a Global Admin directory role to a principal you control
```

## Notes d'exploitation

- `AppRoleAssignment.ReadWrite.All` est auto-escaladant : une application qui le détient s'accorde n'importe quoi, traitez-le donc comme équivalent à Global Admin.
- `RoleManagement.ReadWrite.Directory` attribue directement des rôles d'annuaire, le chemin le plus propre vers la prise de contrôle du tenant depuis une application.
- Elles s'exécutent en tant qu'application sans utilisateur et sans MFA, ce qui explique pourquoi un SP sur-permissionné est un point d'ancrage aussi fort.

## Outils

- **GraphRunner** / **az cli** : énumérer et attribuer les app roles.
- **BARK** (famille `Get-AzureADRoleAssignment`) : identifier les octrois dangereux.

## Références

- [SpecterOps : BARK and Graph permission abuse](https://github.com/BloodHoundAD/BARK)
- [GraphRunner (dafthack)](https://github.com/dafthack/GraphRunner)
- [HackTricks Cloud : Graph permissions](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
