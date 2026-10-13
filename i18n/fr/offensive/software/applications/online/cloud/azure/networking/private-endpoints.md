---
title: "Private endpoints : atteindre le PaaS sur des adresses internes"
order: 3
description: "Abuser d'Azure Private Link et des private endpoints pour atteindre des ressources PaaS sur des adresses internes."
keywords:
  - private endpoint
  - Private Link
  - PaaS
  - accès interne
  - VNet
---

# Private endpoints

Un private endpoint tire une ressource PaaS (un compte de stockage, un Key Vault, un serveur SQL) sur un VNet avec une IP privée via Azure Private Link. La ressource est alors souvent configurée pour refuser l'accès public et ne faire confiance qu'au private endpoint, donc un pied dans la porte à l'intérieur du VNet atteint des ressources invisibles et inaccessibles depuis l'internet. Énumérer les private endpoints cartographie la surface d'attaque interne seulement.

## Énumérer les private endpoints

```bash
az network private-endpoint list \
  --query "[].{Name:name,RG:resourceGroup,Target:privateLinkServiceConnections[0].privateLinkServiceId,IP:customDnsConfigs[0].ipAddresses[0]}" -o table
# the private DNS that resolves the PaaS name to the internal IP
az network private-dns zone list -o table
```

## Notes d'exploitation

- L'IP privée plus la zone DNS privée permettent à une VM compromise de résoudre et d'atteindre `myvault.vault.azure.net` ou `mystorage.blob.core.windows.net` en interne, même quand leur pare-feu public refuse tout, donc le pied dans la porte réseau se substitue à une source en liste d'autorisation.
- L'accès nécessite toujours un credential ou un token pour le service cible ; les private endpoints résolvent l'accessibilité, pas l'autorisation, donc à combiner avec un accès [Key Vault](../credentials/key-vault/index.md) ou [stockage](../storage/index.md).
- Un private endpoint dont la cible est dans un autre abonnement révèle une relation de confiance inter-abonnements qui vaut la peine d'être suivie.

## Outils

- **az network private-endpoint** / **private-dns** : l'inventaire des endpoints et du DNS.
- **Azure Resource Graph** : les private endpoints à travers le tenant.

## Références

- [HackTricks Cloud : réseau Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : Azure Private Link](https://learn.microsoft.com/azure/private-link/private-link-overview)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
