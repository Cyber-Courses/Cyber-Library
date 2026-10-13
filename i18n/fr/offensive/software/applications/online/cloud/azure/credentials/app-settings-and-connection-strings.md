---
title: "App settings et chaînes de connexion : piller la config des Function et Web Apps"
order: 6
description: "Récolter les app settings, les chaînes de connexion et l'environnement Kudu des Function et Web Apps pour y trouver des secrets stockés et des identifiants backend."
keywords:
  - app settings
  - chaînes de connexion
  - Kudu
  - Function App
  - Web App
---

# App settings et chaînes de connexion

Les Function et Web Apps conservent leur configuration sous forme d'**app settings** et de **chaînes de connexion** : mots de passe de base de données, clés de stockage, jetons d'API et la clé hôte de la Function. Ils sont retournés en clair à quiconque dispose d'un accès de gestion à l'application, et sont également lisibles depuis l'environnement Kudu/SCM d'une application en cours d'exécution.

## Lire depuis le plan de gestion

```bash
az webapp config appsettings list --name <app> --resource-group <rg> -o json
az webapp config connection-string list --name <app> --resource-group <rg> -o json
az functionapp config appsettings list --name <fn> --resource-group <rg> -o json
```

## Lire depuis Kudu / l'application en cours d'exécution

Avec un accès de publication ou une exécution de code dans l'application, les mêmes valeurs sont des variables d'environnement, accessibles via la console Kudu ou son API VFS :

```bash
# Kudu runs at https://<app>.scm.azurewebsites.net
curl -s -u '<deploy-user>:<deploy-pass>' \
  https://<app>.scm.azurewebsites.net/api/settings
# or run `env` in the Kudu debug console
```

## Notes d'exploitation

- Les chaînes de connexion contiennent régulièrement des clés de compte de stockage et des identifiants SQL, qui pivotent directement vers le [stockage](../storage/index.md) et les [données](../data/index.md).
- Le paramètre `AzureWebJobsStorage` est la clé du compte de stockage propre à la Function, un accès complet au plan de données de ce compte.
- Les identifiants de publication Kudu eux-mêmes proviennent des [identifiants de déploiement](../serverless/app-service/deployment-credentials.md) de l'application ; l'un alimente l'autre.

## Outils

- **az cli** (`webapp config`, `functionapp config`).
- **MicroBurst** (`Get-AzPasswords`) : extrait les app settings et les identifiants de publication.
- Console **Kudu** / `/api/settings`.

## Références

- [Microsoft : configurer les app settings](https://learn.microsoft.com/azure/app-service/configure-common)
- [HackTricks Cloud : Azure App Service](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
