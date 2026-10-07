---
title: "Key Vault"
order: 2
description: "Reading Azure Key Vault material: secrets, keys, and certificates, reached through data-plane access or an access-policy or RBAC grant."
keywords:
  - Key Vault
  - secrets
  - keys
  - certificates
  - access policy
---

# Key Vault

Azure Key Vault is where applications store their passwords, connection strings, signing keys, and TLS certificates, so a vault you can read is a direct line into the environment's secrets. Access is governed either by legacy **access policies** or by **Azure RBAC**, both on the vault's data plane at `https://<vault>.vault.azure.net`; a managed-identity token for `https://vault.azure.net` or a user with a data-plane role reads it.

## What folds in here

- **[Secrets](secrets.md)**: listing and dumping stored secret values.
- **[Keys and certificates](keys-and-certificates.md)**: extracting or using keys and downloading certificates with their private key.
- **[Access policy](access-policy.md)**: granting yourself vault access when you hold management-plane rights but not data-plane.

## References

- [Microsoft: Key Vault data-plane access](https://learn.microsoft.com/azure/key-vault/general/security-features)
- [HackTricks Cloud: Azure Key Vault](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst (Key Vault)](https://github.com/NetSPI/MicroBurst)
