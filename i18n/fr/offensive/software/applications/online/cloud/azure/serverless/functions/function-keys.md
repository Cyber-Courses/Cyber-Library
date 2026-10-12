---
title: "Function keys : récupérer les host keys et function keys pour invoquer des endpoints protégés"
order: 2
description: "Récupérer les host keys et function keys d'Azure Function pour invoquer des functions protégées et des endpoints d'administration."
keywords:
  - function keys
  - host key
  - master key
  - admin endpoint
  - invoke
---

# Function keys

Les functions dont le niveau d'autorisation est `function` ou `admin` sont protégées par des **clés** : des clés par function, des host keys partagées par l'ensemble de l'application, et la clé `_master` qui atteint l'API d'administration. Avec `Microsoft.Web/sites/host/listkeys` (ou Contributor sur l'application), ces clés sont lisibles, et la master key déverrouille les endpoints d'administration qui invoquent n'importe quelle function et lisent l'état d'exécution.

## Lister les clés

```bash
az functionapp keys list -g <rg> -n <app>            # host and master keys
az functionapp function keys list -g <rg> -n <app> --function-name <fn>
# raw ARM if the CLI verb is unavailable
az rest --method post --uri \
  "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<app>/host/default/listkeys?api-version=2022-03-01"
```

## Invoquer avec une clé récupérée

```bash
curl "https://<app>.azurewebsites.net/api/<fn>?code=<function-key>"
# admin endpoints with the master key
curl -H "x-functions-key: <master-key>" \
  "https://<app>.azurewebsites.net/admin/functions"
```

## Notes d'exploitation

- La clé `_master` équivaut à l'administration de l'application pour le runtime Functions ; traitez sa récupération comme une exécution de code sur l'application.
- Les clés sont également écrites dans le compte de stockage de l'application (conteneur `azure-webjobs-secrets`), de sorte qu'un accès au compte de stockage les livre sans appel ARM.
- Invoquer une function qui porte elle-même une identité managée enchaîne vers l'[identité managée](managed-identity.md).

## Outils

- **Azure CLI** (`functionapp keys list`, `az rest`).
- **MicroBurst** : énumère les Function apps et extrait leurs secrets.

## Références

- [Microsoft: work with access keys in Azure Functions](https://learn.microsoft.com/azure/azure-functions/function-keys-how-to)
- [HackTricks Cloud: Azure App Service and Functions](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
