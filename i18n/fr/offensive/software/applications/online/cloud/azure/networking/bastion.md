---
title: "Bastion : pivoter vers des VM privées"
order: 6
description: "Abuser d'Azure Bastion comme pivot vers des VM privées via RDP et SSH."
keywords:
  - Bastion
  - pivot
  - RDP
  - SSH
  - VM privée
---

# Bastion

Azure Bastion est un hôte de rebond managé qui relaie le RDP et le SSH vers des VM sans IP publique, directement depuis le portail ou la CLI, par le plan de contrôle. Pour un attaquant détenant un token avec des droits sur Bastion et une VM cible, c'est un pivot autorisé vers le réseau privé qui ne nécessite aucune règle NSG entrante et ne laisse à la VM aucune exposition publique à repérer.

## Se connecter via Bastion

```bash
az network bastion list -o table
# tunnel to a private VM's RDP/SSH port through Bastion over the control plane
az network bastion tunnel --name <bastion> -g <rg> \
  --target-resource-id <vm-resource-id> --resource-port 22 --port 50022
# then ssh/rdp to localhost:50022
```

## Notes d'exploitation

- Atteindre une VM de cette manière nécessite une lecture `Microsoft.Network/bastionHosts/*` plus `Reader` sur la VM et l'action de démarrer un tunnel ou une session ; un rôle qui accorde la connexion Bastion est un chemin discret vers des hôtes autrement inaccessibles.
- La connexion est une action de gestion légitime, donc elle se fond dans le trafic d'opérateur normal bien mieux que l'ouverture d'un port NSG.
- Une fois sur la VM, récupérez le token de son [identité managée](../compute/virtual-machines/managed-identity.md) et poursuivez depuis l'hôte.

## Outils

- **az network bastion tunnel / ssh / rdp** : relayer la connexion depuis la CLI.
- **Portail Azure** : le même pivot via la session du navigateur.

## Références

- [HackTricks Cloud : services Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : Azure Bastion](https://learn.microsoft.com/azure/bastion/bastion-overview)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
