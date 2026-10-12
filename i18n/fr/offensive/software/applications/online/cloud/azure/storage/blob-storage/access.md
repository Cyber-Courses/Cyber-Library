---
title: "Accès : lire des blobs via des clés, le RBAC ou l'accès anonyme"
order: 2
description: "Lire et écrire des blobs Azure via des clés de compte, des rôles de données RBAC ou un accès anonyme mal configuré."
keywords:
  - blob access
  - account key
  - RBAC
  - anonymous
  - data plane
---

# Accès

Une fois un conteneur localisé, la lecture de ses blobs dépend de l'autorisation exposée par le compte. Par ordre décroissant de valeur pour l'attaquant : une **clé de compte** récupérée (contrôle total de chaque conteneur, de chaque opération), un **rôle de données Azure RBAC** sur le compte ou le conteneur (Storage Blob Data Reader ou Contributor), ou un accès **anonyme** là où le niveau public l'autorise.

## Avec une clé de compte

La clé de compte est le credential maître du plan de données. Récupérez-la depuis le plan de gestion (si vous disposez de `Microsoft.Storage/storageAccounts/listkeys/action`) ou depuis une configuration divulguée :

```bash
az storage account keys list --account-name acme -g rg --query '[0].value' -o tsv
# then read with the key
az storage blob list   --account-name acme --account-key <KEY> --container-name backups
az storage blob download --account-name acme --account-key <KEY> -c backups -n secrets.tar -f ./secrets.tar
```

## Avec un rôle de données RBAC

Un principal détenant un rôle Storage Blob Data lit sans la clé, à l'aide de son propre jeton :

```bash
az storage blob download --account-name acme --auth-mode login \
  -c backups -n secrets.tar -f ./secrets.tar
```

Notez que les rôles du plan de gestion tels que Contributor n'accordent pas la lecture du plan de données par défaut, mais qu'ils peuvent lire la clé de compte (`listkeys`), ce qui la permet : Contributor sur un compte de stockage équivaut donc à un accès complet aux données.

## Anonyme

```bash
curl -O "https://acme.blob.core.windows.net/backups/secrets.tar"
```

## Notes d'exploitation

- `listkeys` est l'escalade discrète : un rôle du plan de gestion capable de lire les clés contourne entièrement le RBAC du plan de données.
- Examinez les quatre services du compte (blob, file, queue, table) avec la même clé ; les opérateurs oublient souvent que la clé déverrouille chacun d'eux.
- Storage Explorer et `azcopy` avec la clé ou une SAS extraient des conteneurs entiers rapidement pour un tri hors ligne.

## Outils

- **az CLI** / **azcopy** : lectures par clé, SAS et `--auth-mode login`, et copie en masse.
- **Azure Storage Explorer** : navigation en pointer-cliquer avec une clé ou une SAS.
- **MicroBurst** (`Get-AzStorageKeysREST`) : extraction des clés de compte via REST à partir d'un jeton.

## Références

- [HackTricks Cloud: Azure storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [Microsoft: authorize access to blobs](https://learn.microsoft.com/azure/storage/common/authorize-data-access)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
