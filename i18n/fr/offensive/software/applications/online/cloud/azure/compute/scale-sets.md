---
title: "Scale sets : run command et extensions sur chaque instance"
order: 5
description: "Exécuter des commandes et des extensions sur un Azure VM Scale Set pour atteindre chaque instance et son identité managée."
keywords:
  - VMSS
  - scale sets
  - run command
  - extension
  - identité managée
---

# Scale sets

Un VM Scale Set (VMSS) est une flotte de VM identiques gérées comme une seule ressource. Les mêmes droits de run command et d'extension qui permettent de prendre le contrôle d'une VM unique s'appliquent à l'ensemble, de sorte qu'une seule action atteint chaque instance, et l'**identité managée** de l'ensemble est partagée par toutes.

## Exécuter sur toute la flotte

```bash
az vmss run-command invoke -g <rg> -n <vmss> --instance-id '*' \
  --command-id RunShellScript --scripts "id"
# or install an extension on the model so new instances inherit it
az vmss extension set -g <rg> --vmss-name <vmss> \
  --name CustomScript --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"..."}'
```

## Notes d'exploitation

- Définir une extension sur le **modèle** du VMSS signifie que chaque instance, y compris celles ajoutées plus tard par scale out, l'exécute : une exécution durable sur toute la flotte.
- Le jeton de l'identité managée du scale set est accessible depuis n'importe quelle instance via IMDS, comme pour une [VM](virtual-machines/managed-identity.md) autonome.

## Outils

- **Azure CLI** (`az vmss run-command invoke`, `az vmss extension set`).
- **MicroBurst** : exécution de commandes en masse.

## Références

- [Microsoft : extensions VMSS](https://learn.microsoft.com/azure/virtual-machine-scale-sets/overview)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
