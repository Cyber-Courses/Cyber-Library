---
title: "Activity Log : contourner et assécher le registre des opérations de l'abonnement"
description: "Contourner ou neutraliser l'Azure Activity Log et les registres d'opérations au niveau de l'abonnement."
keywords:
  - Activity Log
  - registre d'opérations
  - abonnement
  - évasion de détection
  - anti-forensique
---

# Activity Log

L'Activity Log est le registre, au niveau de l'abonnement, des opérations du plan de contrôle (qui a appelé quelle écriture ARM, sur quelle ressource). Il ne peut être ni supprimé ni modifié et est conservé 90 jours, donc il ne s'aveugle pas directement. Il s'aveugle indirectement : en coupant les **diagnostic settings** qui le copient dans un workspace Log Analytics, un Event Hub ou un compte de stockage où résident réellement un SIEM et une rétention longue durée, et en privilégiant les opérations que le log ne capture jamais.

## Voir d'abord ce qu'il capture

```bash
# recent control-plane operations (recon your own noise before acting)
az monitor activity-log list --offset 2h --query "[].{op:operationName.value,caller:caller,status:status.value}" -o table
```

## Couper l'export vers les sinks en aval

L'Activity Log reste en place, mais on arrête son flux vers le pipeline de logs de l'abonnement :

```bash
# list subscription-level Activity Log exports, then remove them
az monitor diagnostic-settings subscription list
az monitor diagnostic-settings subscription delete --name <setting>
```

## Notes d'exploitation

- Le log n'enregistre que les écritures du plan de contrôle ARM. Les actions du **plan de données** (lire un blob, récupérer un secret Key Vault via son point de terminaison de données, interroger une base) n'y apparaissent pas, donc la tradecraft du plan de données est discrète par construction ; voir les chemins du plan de données sous [credentials](../credentials/index.md) et [stockage](../storage/index.md).
- Un `DELETE` sur une diagnostic setting d'abonnement est lui-même un événement de l'Activity Log, donc il atterrit dans le stockage de 90 jours ; il ne fait que couper la copie SIEM en aval, il n'efface pas l'enregistrement natif.
- L'alerting s'appuie généralement sur la copie exportée dans Log Analytics, donc couper l'export est ce qui arrête réellement le déclenchement des règles ; à combiner avec le démantèlement des diagnostic settings au scope de la ressource via [Azure Monitor](azure-monitor.md).

## Outils

- **Azure CLI** (`az monitor activity-log`, `az monitor diagnostic-settings subscription`).
- **MicroBurst** : énumération de l'abonnement pour cartographier où les logs sont expédiés.

## Références

- [HackTricks Cloud : surveillance et journalisation Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Stratus Red Team : techniques Azure](https://stratus-red-team.cloud/attack-techniques/azure/)
- [Microsoft : Azure Activity Log](https://learn.microsoft.com/azure/azure-monitor/essentials/activity-log)
