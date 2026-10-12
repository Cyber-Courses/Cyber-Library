---
title: "Network security groups : exposer et atteindre des ressources filtrées"
order: 1
description: "Lire et réécrire les règles des network security groups Azure pour exposer ou atteindre des ressources autrement filtrées."
keywords:
  - NSG
  - network security group
  - règle de pare-feu
  - exposition
  - entrant
---

# Network security groups

Un network security group (NSG) est une liste d'autorisation avec état de règles entrantes et sortantes, liée à un sous-réseau ou à une NIC. La défaillance courante est une règle entrante avec pour source `Internet` ou `*` sur un port de gestion ou de base de données, mettant RDP, SSH ou une base de données sur l'internet public. Avec un accès reader, vous lisez chaque NSG et résolvez quelles ressources il protège ; avec `Microsoft.Network/networkSecurityGroups/securityRules/write`, vous ajoutez votre propre règle d'autorisation pour atteindre quelque chose auparavant filtré.

## Trouver les règles entrantes grandes ouvertes

```bash
# NSGs with an Allow inbound rule open to the internet
az network nsg list --query "[].{Name:name,RG:resourceGroup}" -o table
az network nsg rule list --nsg-name <nsg> -g <rg> \
  --query "[?direction=='Inbound' && access=='Allow' && (sourceAddressPrefix=='*' || sourceAddressPrefix=='Internet')].{Name:name,Port:destinationPortRange,Src:sourceAddressPrefix}" -o table
```

## Ouvrir un chemin que vous contrôlez

```bash
# add an inbound allow rule (needs securityRules/write on the NSG)
az network nsg rule create -g <rg> --nsg-name <nsg> -n allow-attacker \
  --priority 100 --direction Inbound --access Allow --protocol Tcp \
  --source-address-prefixes <your-ip>/32 --destination-port-ranges 3389 22 1433
```

## Notes d'exploitation

- Les ports `3389`, `22`, `1433`, `3306`, `5432`, `6379` et `27017` ouverts à `Internet` sont les trouvailles de grande valeur : RDP, SSH et des datastores faiblement authentifiés.
- Les règles NSG s'appliquent au niveau du sous-réseau et de la NIC ; une règle permissive de sous-réseau peut être défaite par une règle NIC plus stricte et inversement, donc vérifiez les deux.
- L'ajout d'une règle est journalisé dans l'Activity Log ; un numéro de priorité plus bas l'emporte, donc une nouvelle règle en priorité 100 l'emporte sur un deny en 200 sans le supprimer.

## Outils

- **L'équivalent de l'AWS CLI est `az network nsg`** : énumération et écriture de règles.
- **MicroBurst** / **Azure Resource Graph** : posture NSG à l'échelle de l'abonnement.
- **nmap** / **masscan** : confirmer l'accessibilité des ports exposés depuis l'extérieur.

## Références

- [HackTricks Cloud : réseau Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : network security groups](https://learn.microsoft.com/azure/virtual-network/network-security-groups-overview)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
