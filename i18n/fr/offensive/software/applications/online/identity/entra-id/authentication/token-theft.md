---
title: "Vol de jetons : récupération et rejeu de jetons Entra"
order: 7
description: "Récupération et rejeu de jetons d'accès et d'actualisation Entra : extraction depuis le disque, le navigateur et le cache de jetons, puis rejeu avec TokenTactics et ROADtools."
keywords:
  - token theft
  - refresh token
  - access token
  - token replay
  - TokenTactics
---

# Vol de jetons

L'authentification Entra produit des jetons porteurs (bearer tokens), et un jeton porteur suffit à lui seul : il suffit de voler un jeton d'accès ou d'actualisation sur un hôte et de le rejouer, sans mot de passe ni MFA. Les jetons d'actualisation sont la cible la plus précieuse, car ils génèrent de nouveaux jetons d'accès, souvent sur plusieurs ressources internes (first-party).

## Où résident les jetons et comment les rejouer

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

## Notes d'exploitation

- Un jeton d'actualisation issu d'un client interne à large portée (par exemple l'Az CLI) peut être échangé sur Graph, Azure management et d'autres ressources, ce qui élargit la portée d'un seul vol.
- Les jetons conservent les revendications (claims) d'origine de MFA et de périphérique, de sorte que le rejeu en hérite et passe l'accès conditionnel qui fait confiance à ces revendications.
- Les jetons d'accès ont une durée de vie courte (environ une heure) ; récupérez le jeton d'actualisation pour la persistance.

## Outils

- **TokenTactics** (`Invoke-RefreshTo*`) : échange de jetons d'actualisation entre ressources.
- **ROADtools** (`roadtx`) : gestion et actualisation des jetons.
- **AADInternals** : extraction du cache et utilisation des jetons.

## Références

- [TokenTactics (rvrsh3ll)](https://github.com/rvrsh3ll/TokenTactics)
- [ROADtools](https://github.com/dirkjanm/ROADtools)
- [HackTricks Cloud : abus de jetons](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
