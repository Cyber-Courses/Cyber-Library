---
title: "Azure credentials"
description: "Harvesting Azure credentials: managed-identity tokens from IMDS, Key Vault secrets and keys, storage account keys, Automation Account assets, and app settings and connection strings."
keywords:
  - Azure credentials
  - IMDS
  - Key Vault
  - storage keys
  - Automation assets
---

# Credentials

Every ARM and data-plane call needs a token or key, so harvesting them is how access widens. Azure credentials arrive as **managed-identity tokens** minted from the instance metadata service, user and service-principal **access and refresh tokens**, **storage account keys** that unlock a whole account's data plane, and the secrets that services like Key Vault, Automation Accounts, and App Service hand back when read.

## What folds in here

- **[Instance metadata](instance-metadata/index.md)**: minting a resource's managed-identity token from IMDS, directly or through SSRF.
- **[Key Vault](key-vault/index.md)**: secrets, keys, and certificates over the vault data plane.
- **[Storage keys](storage-keys.md)**: `listKeys` to full data-plane access over an account.
- **[Access tokens](access-tokens.md)**: cached `az` and MSAL tokens looted from disk.
- **[Automation assets](automation-assets.md)**: Automation Account credential, variable, and connection assets.
- **[App settings and connection strings](app-settings-and-connection-strings.md)**: Function and Web App config secrets.

The IMDS pages are the Azure end of the web [server-side request forgery](../../../../server/web/code/injection/request-forgery/index.md) technique, cross-referenced rather than duplicated.

## References

- [HackTricks Cloud: Azure credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst (Get-AzPasswords)](https://github.com/NetSPI/MicroBurst)
- [Microsoft: managed identities and IMDS](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token)
