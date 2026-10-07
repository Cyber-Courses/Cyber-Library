---
title: "Managed identity: minting the App Service token"
order: 3
description: "Minting the App Service managed-identity token from the app's local identity endpoint."
keywords:
  - App Service managed identity
  - identity endpoint
  - token
  - IDENTITY_ENDPOINT
---

# Managed identity

An App Service app with a managed identity exposes a local token endpoint through `IDENTITY_ENDPOINT` and `IDENTITY_HEADER`, reachable only from inside the app. Code execution on the app, through [Kudu](kudu-and-scm.md) or a deployed [web shell](deployment-credentials.md), mints the identity's token and continues as that principal against ARM, Key Vault, or Graph.

## Minting the token

```bash
# run inside the app (Kudu console or deployed handler)
curl -s "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2019-08-01" \
  -H "X-IDENTITY-HEADER: $IDENTITY_HEADER"
# change resource to https://vault.azure.net or https://graph.microsoft.com to pivot
```

## Exploitation notes

- The endpoint is not the VM IMDS (`169.254.169.254`); App Service injects its own per-app endpoint and header, so both env vars are required.
- The identity's role assignments decide the blast radius; enumerate them and chain into [identity](../../identity/managed-identities/index.md).
- A user-assigned identity bound to several apps turns one app compromise into an identity reused elsewhere.

## Tools

- **curl** against `IDENTITY_ENDPOINT` from the app context.
- **MicroBurst** (`Get-AzPasswords`): extracts App Service identity tokens.

## References

- [Microsoft: managed identity in App Service](https://learn.microsoft.com/azure/app-service/overview-managed-identity)
- [HackTricks Cloud: Azure managed identities](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps: Managed Identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
