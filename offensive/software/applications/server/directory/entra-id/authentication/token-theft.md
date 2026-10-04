---
title: "Token theft: harvesting and replaying Entra tokens"
description: "Harvesting and replaying Entra access and refresh tokens: extraction from disk, browser, and the token cache, and replay with TokenTactics and ROADtools."
keywords:
  - token theft
  - refresh token
  - access token
  - token replay
  - TokenTactics
---

# Token theft

Entra authentication produces bearer tokens, and a bearer token is enough on its own: steal an access or refresh token from a host and replay it, no password or MFA required. Refresh tokens are the prize because they mint fresh access tokens, often across multiple first-party resources.

## Where tokens live and how to replay

```bash
# token caches on disk / in memory
#   Az CLI:    ~/.azure/msal_token_cache.json / accessTokens.json
#   Az PowerShell: TokenCache.dat
#   browser:   ESTSAUTH / ESTSAUTHPERSISTENT cookies, and the token cache of Teams/Office
```

```powershell
# replay a stolen refresh token for new access tokens with TokenTactics
Invoke-RefreshToMSGraphToken -refreshToken <rt> -domain example.com
Invoke-RefreshToAzureManagementToken -refreshToken <rt> -domain example.com
```

## Exploitation notes

- A refresh token from a broad first-party client (for example the Az CLI) exchanges across Graph, Azure management, and other resources, widening a single theft.
- Tokens carry the original MFA and device claims, so replay inherits them and passes conditional access that trusts those claims.
- Access tokens are short-lived (roughly an hour); grab the refresh token for persistence.

## Tools

- **TokenTactics** (`Invoke-RefreshTo*`): refresh-token exchange across resources.
- **ROADtools** (`roadtx`): token handling and refresh.
- **AADInternals**: cache extraction and token use.

## References

- [TokenTactics (rvrsh3ll)](https://github.com/rvrsh3ll/TokenTactics)
- [ROADtools](https://github.com/dirkjanm/ROADtools)
- [HackTricks Cloud: token abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
