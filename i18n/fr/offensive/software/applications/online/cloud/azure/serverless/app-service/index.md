---
title: "App Service"
order: 4
description: "Attaquer les applications web Azure App Service : la console Kudu et SCM, les identifiants de déploiement et l'identité managée de l'application."
keywords:
  - App Service
  - Kudu
  - SCM
  - deployment credentials
  - managed identity
---

# App Service

Une application web App Service expose le site de gestion **Kudu/SCM** (une console web et une API de fichiers), est livrée avec des **identifiants de publication** qui poussent du code, et peut porter une **identité managée**. Chacun de ces trois éléments donne une exécution de code ou un jeton d'identité : Kudu est un shell, les identifiants de publication déploient un web shell, et l'identité managée est le butin qui se cache derrière les deux.

## Ce qui se rattache ici

- **[Kudu et SCM](kudu-and-scm.md)** : la console de gestion pour un web shell, l'accès aux fichiers et les secrets d'environnement.
- **[Identifiants de déploiement](deployment-credentials.md)** : les profils de publication qui poussent du code et lisent la configuration.
- **[Identité managée](managed-identity.md)** : forger le jeton d'identité managée de l'application.

## Références

- [HackTricks Cloud: Azure App Service](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Kudu service overview](https://learn.microsoft.com/azure/app-service/resources-kudu)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
