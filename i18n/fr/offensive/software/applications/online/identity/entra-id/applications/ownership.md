---
title: "Propriété : prendre le contrôle d'une application via son propriétaire"
order: 6
description: "Abus de la propriété d'applications et de principaux de service : un propriétaire peut ajouter des identifiants et modifier la configuration pour prendre le contrôle de l'identité de l'application."
keywords:
  - propriété d'application
  - propriétaire
  - principal de service
  - identifiants
  - prise de contrôle
---

# Propriété

Le **propriétaire** d'une application ou d'un principal de service peut modifier l'objet, ce qui inclut l'ajout d'identifiants. La propriété d'une application privilégiée équivaut donc à une prise de contrôle totale : ajoutez un secret, authentifiez-vous en tant qu'application et héritez de ses autorisations. La propriété est attribuée à la légère et rarement revue, ce qui en fait un accès discret et courant vers une application puissante.

## Abuser de la propriété

```bash
# list apps you own (or add yourself as owner where you hold the rights)
az ad app owner list --id <appId>
az ad app owner add --id <appId> --owner-object-id <you>

# as owner, add a credential and log in as the app
az ad app credential reset --id <appId> --append
```

## Notes d'exploitation

- La propriété combinée aux [identifiants d'application](application-credentials.md) forme la chaîne : le propriétaire ajoute un secret, puis s'authentifie en tant qu'application.
- Ciblez les propriétaires d'applications détenant des rôles d'application Graph dangereux ; le lien de propriété se transforme en ces autorisations.
- S'ajouter soi-même comme propriétaire (là où un rôle ou une autorisation le permet) constitue en soi une persistance, réattribuable après le nettoyage d'un identifiant.

## Outils

- **az cli** (`az ad app owner`).
- **BARK** / **AzureHound** : chemins d'attaque par lien de propriété.

## Références

- [SpecterOps : liens de propriété AzureHound](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud : propriété d'application](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : propriétaires d'enregistrements d'applications](https://learn.microsoft.com/entra/identity-platform/app-objects-and-service-principals)
