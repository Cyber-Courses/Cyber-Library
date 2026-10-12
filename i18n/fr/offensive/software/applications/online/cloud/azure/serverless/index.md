---
title: "Azure serverless"
order: 5
description: "Attaquer le serverless et l'automatisation Azure : identités managées de Function et Logic App, runbooks et RunAs des Automation Accounts, Kudu et déploiement App Service, et Deployment Scripts."
keywords:
  - Azure serverless
  - Functions
  - Logic Apps
  - Automation Accounts
  - App Service
---

# Serverless

Les services serverless et d'automatisation d'Azure exécutent tous du code, et la quasi-totalité d'entre eux l'exécutent sous une **identité managée** ou un credential stocké : chacun est donc un chemin du contrôle du service vers l'identité qu'il porte. Le schéma est le même partout : faire exécuter du code (ou déclencher un workflow), puis forger le jeton du service et poursuivre en tant que lui.

Les Automation Accounts et App Service sont les cibles les plus riches : les runbooks Automation s'exécutent en tant qu'identité privilégiée et détiennent des assets de credential en clair, et App Service expose la console Kudu et les identifiants de publication.

## Ce qui se rattache ici

- **[Functions](functions/index.md)** : s'exécuter en tant qu'identité managée de la Function app et récupérer les function keys et host keys.
- **[Logic Apps](logic-apps.md)** : ajouter une étape de workflow qui appelle ARM ou Graph en tant qu'identité managée du workflow.
- **[Automation Accounts](automation-accounts/index.md)** : runbooks, compte RunAs et workers hybrides pour l'exécution en tant qu'identité privilégiée.
- **[App Service](app-service/index.md)** : la console Kudu et SCM, les identifiants de déploiement et l'identité managée de l'application.
- **[Deployment Scripts](deployment-scripts.md)** : les `deploymentScripts` ARM qui exécutent un conteneur en tant qu'identité affectée par l'utilisateur choisie.

L'**endpoint de jeton** de l'identité managée lui-même est traité sous [credentials](../credentials/instance-metadata/index.md) ; le cadrage en termes d'escalade de privilèges du rattachement d'une identité se trouve dans [identité](../identity/privilege-escalation/index.md).

## Références

- [HackTricks Cloud: Azure services](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [PowerZure](https://github.com/hausec/PowerZure)
