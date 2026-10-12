---
title: "Jetons d'accès : piller les jetons az et MSAL en cache"
order: 4
description: "Piller les jetons d'accès et de rafraîchissement Azure en cache depuis le répertoire .azure de l'az cli, les caches MSAL et les brokers de jetons."
keywords:
  - jeton d'accès
  - jeton de rafraîchissement
  - az cli
  - .azure
  - MSAL
---

# Jetons d'accès

Un hôte compromis sur lequel quelqu'un a exécuté `az`, Azure PowerShell ou un SDK Azure détient généralement des **jetons d'accès et de rafraîchissement** en cache. Le jeton de rafraîchissement est le butin : il génère de nouveaux jetons d'accès pour toute ressource à laquelle l'utilisateur a consenti, longtemps après la session, et survit souvent à une réinitialisation de mot de passe.

## Où vivent les jetons

```bash
# az cli
~/.azure/msal_token_cache.json          # MSAL cache: access + refresh tokens
~/.azure/azureProfile.json              # subscriptions, tenant, signed-in identity
# legacy
~/.azure/accessTokens.json

# Azure PowerShell
~/.Azure/AzureRmContext.json
```

## Utiliser ce que vous trouvez

```bash
# a live az session: mint a token for any audience
az account get-access-token --resource https://graph.microsoft.com/ --query accessToken -o tsv
az account get-access-token --resource https://management.azure.com/ --query accessToken -o tsv
```

Un **jeton de rafraîchissement** récupéré est rejoué avec ROADtools ou TokenTactics pour demander des jetons d'accès pour des client IDs et des audiences first-party arbitraires, y compris ceux que la victime n'a jamais utilisés de façon interactive (jetons de rafraîchissement de la famille FOCI).

## Notes d'exploitation

- Préférez le jeton de rafraîchissement au jeton d'accès : les jetons d'accès expirent en une heure environ, les jetons de rafraîchissement durent de quelques jours à quelques mois et se régénèrent librement.
- Les jetons de rafraîchissement FOCI (« family of client IDs ») fonctionnent sur de nombreux clients first-party Microsoft, de sorte qu'un seul jeton de rafraîchissement pivote entre les points de terminaison Graph, ARM et Office.
- Les jetons sont des identifiants bearer sans liaison d'appareil, à moins que CAE ou la protection de jetons ne soit imposée, de sorte qu'ils se déplacent proprement vers l'hôte de l'attaquant.

## Outils

- **ROADtools** (`roadrecon`, `roadtx`) : importer et échanger des jetons de rafraîchissement entre clients.
- **TokenTactics** : demander des jetons pour différentes ressources à partir d'un jeton de rafraîchissement.
- **az cli** (`account get-access-token`) sur une session active.

## Références

- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
- [TokenTactics (rvrsh3ll)](https://github.com/rvrsh3ll/TokenTactics)
- [HackTricks Cloud : vol de jetons Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
