---
title: "Keys and certificates: extracting and using Key Vault keys"
order: 2
description: "Extracting or using Key Vault keys and certificates for decryption, signing, and impersonation."
keywords:
  - Key Vault
  - keys
  - certificates
  - signing
  - decryption
---

# Keys and certificates

Beyond secrets, a vault holds cryptographic **keys** (for signing and decryption) and **certificates** (often with the private key). A certificate stored in Key Vault is also exposed as a secret, so `get` on the secret backing it returns the full PFX, private key included, which is an immediate path to impersonating whatever the certificate authenticates.

## Downloading a certificate with its private key

```bash
# the certificate's private material is the secret of the same name
az keyvault secret show --vault-name <vault> --name <cert> --query value -o tsv | base64 -d > cert.pfx
az keyvault certificate list --vault-name <vault> --query '[].id' -o tsv
```

A PFX for an app or service-principal certificate lets you authenticate as that principal (for example `az login --service-principal --certificate`).

## Using keys without extracting them

Non-exportable keys still perform operations server-side for anyone with `sign`/`decrypt`/`unwrapKey`:

```bash
az keyvault key list --vault-name <vault> --query '[].kid' -o tsv
# sign or decrypt as the key without ever seeing it
az keyvault key sign --vault-name <vault> --name <key> --algorithm RS256 --value <base64-digest>
az keyvault key decrypt --vault-name <vault> --name <key> --algorithm RSA-OAEP --value <base64-ct>
```

## Exploitation notes

- A certificate that backs a service-principal or app login is the prize: the PFX is a durable credential surviving until the cert is rotated.
- `decrypt`/`unwrapKey` on a data-encryption key can unlock storage or database contents encrypted with customer-managed keys.
- Operations are logged on the vault, but holding the extracted PFX moves the attacker off the vault entirely.

## Tools

- **az cli** (`keyvault key`, `keyvault certificate`).
- **MicroBurst** (`Get-AzPasswords -ExportCerts`): export certificates with private keys.

## References

- [Microsoft: Key Vault keys](https://learn.microsoft.com/azure/key-vault/keys/about-keys)
- [HackTricks Cloud: Azure Key Vault](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
