---
title: "Managed identity: minting the function app's token"
description: "Executing in an Azure Function to mint and use the function app's managed-identity token."
keywords:
  - Function managed identity
  - token
  - IMDS
  - privilege escalation
---

# Managed identity

A Function app with a system- or user-assigned **managed identity** exposes a local token endpoint to the running code through the `IDENTITY_ENDPOINT` and `IDENTITY_HEADER` environment variables. Any execution inside the function reads those and mints a token for ARM, Graph, Key Vault, or storage, then acts as the identity, whose role assignments are frequently broader than the app warrants.

## Minting the token from inside the function

```bash
# App Service / Functions managed-identity endpoint (not 169.254.169.254)
curl -s "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2019-08-01" \
  -H "X-IDENTITY-HEADER: $IDENTITY_HEADER"
# swap resource for https://vault.azure.net or https://graph.microsoft.com to pivot
```

Export the returned `access_token` and call ARM directly:

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://management.azure.com/subscriptions?api-version=2020-01-01"
```

## Exploitation notes

- To get execution in the first place, deploy or overwrite code (see [deployment credentials](../app-service/deployment-credentials.md) and [Kudu](../app-service/kudu-and-scm.md)), or use an application injection in the hosted code.
- The token's power is the identity's role assignments; enumerate them with the token and chain into [identity](../../identity/managed-identities/index.md).
- A user-assigned identity shared across apps means one compromised function yields an identity used elsewhere.

## Tools

- **curl** against `IDENTITY_ENDPOINT`.
- **MicroBurst** (`Get-AzPasswords`): pulls managed-identity tokens and app secrets at scale.

## References

- [HackTricks Cloud: Azure managed identities](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: managed identity token endpoint for App Service and Functions](https://learn.microsoft.com/azure/app-service/overview-managed-identity)
- [SpecterOps: Managed Identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
