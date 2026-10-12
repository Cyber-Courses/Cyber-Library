---
title: "Identifiants d'application : ajouter un secret ou un certificat"
order: 4
description: "Ajouter des secrets ou des certificats à une application ou un service principal où vous pouvez écrire, puis s'authentifier en tant que cette identité avec ses permissions Graph."
keywords:
  - identifiants d'application
  - addPassword
  - addKey
  - client secret
  - certificat
---

# Identifiants d'application

Si vous pouvez écrire sur une application ou un service principal (en tant que propriétaire, ou avec le bon rôle d'annuaire ou la bonne permission Graph), vous ajoutez votre propre **client secret ou certificat**, puis vous vous authentifiez en tant que cette identité. Vous héritez de tout ce qu'elle détient, et l'identifiant persiste indépendamment de tout utilisateur.

## Ajouter un identifiant et se connecter

```bash
# add a client secret to an app registration
az ad app credential reset --id <appId> --append
# or via Graph: POST /applications/{id}/addPassword  (or addKey for a certificate)

# authenticate as the application
az login --service-principal -u <appId> -p <secret> --tenant <tenant>
```

## Notes d'exploitation

- C'est l'élévation standard après toute primitive d'écriture d'application (propriété, rôle Application Administrator, ou permission Graph `Application.ReadWrite`).
- Un secret ou un certificat ajouté est une persistance durable : il survit aux réinitialisations de mot de passe des utilisateurs et passe facilement inaperçu parmi les identifiants d'application légitimes.
- Visez les applications qui détiennent déjà des app roles Graph privilégiés afin que le nouvel identifiant en hérite ([permissions d'API](api-permissions.md)).

## Outils

- **az cli** (`az ad app credential reset`).
- **BARK** (`New-AppRegSecret` / `New-SPSecret`) : automatiser l'ajout d'identifiants.

## Références

- [SpecterOps : BARK](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud : application credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : application and service principal objects](https://learn.microsoft.com/entra/identity-platform/app-objects-and-service-principals)
