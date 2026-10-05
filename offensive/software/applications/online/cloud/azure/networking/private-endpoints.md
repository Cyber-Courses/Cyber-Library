---
title: "Private endpoints: reaching PaaS over internal addresses"
description: "Abusing Azure Private Link and private endpoints to reach PaaS resources over internal addresses."
keywords:
  - private endpoint
  - Private Link
  - PaaS
  - internal access
  - VNet
---

# Private endpoints

A private endpoint pulls a PaaS resource (a storage account, Key Vault, SQL server) onto a VNet with a private IP through Azure Private Link. The resource is often then configured to deny public access and trust only the private endpoint, so a foothold inside the VNet reaches resources that are invisible and unreachable from the internet. Enumerating private endpoints maps the internal-only attack surface.

## Enumerating private endpoints

```bash
az network private-endpoint list \
  --query "[].{Name:name,RG:resourceGroup,Target:privateLinkServiceConnections[0].privateLinkServiceId,IP:customDnsConfigs[0].ipAddresses[0]}" -o table
# the private DNS that resolves the PaaS name to the internal IP
az network private-dns zone list -o table
```

## Exploitation notes

- The private IP plus the private DNS zone let a compromised VM resolve and reach `myvault.vault.azure.net` or `mystorage.blob.core.windows.net` internally even when their public firewall denies all, so the network foothold substitutes for an allow-listed source.
- Access still needs a credential or token for the target service; private endpoints solve reachability, not authorization, so pair with [Key Vault](../credentials/key-vault/index.md) or [storage](../storage/index.md) access.
- A private endpoint whose target is in another subscription reveals a cross-subscription trust worth following.

## Tools

- **az network private-endpoint** / **private-dns**: the endpoint and DNS inventory.
- **Azure Resource Graph**: private endpoints across the tenant.

## References

- [HackTricks Cloud: Azure networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Azure Private Link](https://learn.microsoft.com/azure/private-link/private-link-overview)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
