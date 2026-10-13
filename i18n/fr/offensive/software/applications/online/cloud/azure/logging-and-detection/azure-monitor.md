---
title: "Azure Monitor : supprimer les diagnostic settings et voler les clés de workspace"
description: "Altérer Azure Monitor et Log Analytics : désactiver les diagnostic settings et voler les clés partagées du workspace."
keywords:
  - Azure Monitor
  - Log Analytics
  - diagnostic settings
  - clé de workspace
  - altération des logs
---

# Azure Monitor

Azure Monitor est le tuyau qui achemine les logs et métriques des ressources vers un workspace **Log Analytics**, où Sentinel et les règles d'alerte les lisent. Deux mouvements offensifs comptent : supprimer les **diagnostic settings** d'une ressource pour que ses logs cessent de circuler, et voler les **clés partagées** du workspace pour pouvoir forger ou noyer l'ingestion.

## Arrêter les logs à la ressource

```bash
# find what a resource ships and where, then remove it
az monitor diagnostic-settings list --resource <resource-id> -o table
az monitor diagnostic-settings delete --name <setting> --resource <resource-id>
```

Sans diagnostic setting, la ressource continue de fonctionner mais ses logs n'atteignent jamais le workspace, donc rien en aval ne peut déclencher d'alerte à leur sujet.

## Voler les clés du workspace

```bash
# primary/secondary shared keys for the Log Analytics workspace
az monitor log-analytics workspace get-shared-keys -g <rg> -n <workspace>
```

La clé partagée autorise l'ingestion via l'API HTTP Data Collector : avec elle, vous pouvez injecter des enregistrements fabriqués pour noyer les vrais événements dans le bruit, ou simuler une activité bénigne.

## Notes d'exploitation

- Les diagnostic settings sont par ressource, donc aveugler une VM ou un Key Vault ne touche pas le reste ; énumérez l'ensemble complet avant de compter sur le silence.
- Les Data Collection Rules (DCR) alimentent le pipeline plus récent d'Azure Monitor Agent ; supprimez ou détachez la DCR (`az monitor data-collection rule`) en plus des diagnostic settings classiques.
- La clé de workspace est un secret à longue durée de vie ; c'est aussi un point d'ancrage de persistance pour continuer l'injection de logs après la perte des autres accès.
- Supprimer une setting est un événement de l'Activity Log (voir [Activity Log](activity-log.md)) ; cela coupe le flux SIEM mais est lui-même enregistré.

## Outils

- **Azure CLI** (`az monitor diagnostic-settings`, `az monitor log-analytics workspace get-shared-keys`, `az monitor data-collection rule`).
- **MicroBurst** : énumération des workspaces et des clés à l'échelle d'un abonnement.

## Références

- [HackTricks Cloud : surveillance Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
- [Microsoft : API data collector de Log Analytics](https://learn.microsoft.com/azure/azure-monitor/logs/data-collector-api)
