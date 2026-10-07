---
title: "Access tokens: looting cached az and MSAL tokens"
order: 4
description: "Looting cached Azure access and refresh tokens from the az cli .azure directory, MSAL caches, and token brokers."
keywords:
  - access token
  - refresh token
  - az cli
  - .azure
  - MSAL
---

# Access tokens

A compromised host where someone has run `az`, Azure PowerShell, or an Azure SDK usually holds cached **access and refresh tokens**. A refresh token is the prize: it mints fresh access tokens for any resource the user consented to, long after the session, and often survives a password reset.

## Where the tokens live

```bash
# az cli
~/.azure/msal_token_cache.json          # MSAL cache: access + refresh tokens
~/.azure/azureProfile.json              # subscriptions, tenant, signed-in identity
# legacy
~/.azure/accessTokens.json

# Azure PowerShell
~/.Azure/AzureRmContext.json
```

## Using what you find

```bash
# a live az session: mint a token for any audience
az account get-access-token --resource https://graph.microsoft.com/ --query accessToken -o tsv
az account get-access-token --resource https://management.azure.com/ --query accessToken -o tsv
```

A recovered **refresh token** is replayed with ROADtools or TokenTactics to request access tokens for arbitrary first-party client IDs and audiences, including ones the victim never used interactively (FOCI family refresh tokens).

## Exploitation notes

- Prefer the refresh token over an access token: access tokens expire in about an hour, refresh tokens last days to months and re-mint freely.
- FOCI ("family of client IDs") refresh tokens work across many Microsoft first-party clients, so one refresh token pivots between Graph, ARM, and Office endpoints.
- Tokens are bearer credentials with no device binding unless CAE or token-protection is enforced, so they move to the attacker's host cleanly.

## Tools

- **ROADtools** (`roadrecon`, `roadtx`): import and exchange refresh tokens across clients.
- **TokenTactics**: request tokens for different resources from a refresh token.
- **az cli** (`account get-access-token`) on a live session.

## References

- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
- [TokenTactics (rvrsh3ll)](https://github.com/rvrsh3ll/TokenTactics)
- [HackTricks Cloud: Azure token theft](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
