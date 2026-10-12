---
title: "Clés de stockage : listKeys pour un accès complet au plan de données"
order: 3
description: "Récupérer les clés de compte de stockage Azure via listKeys pour obtenir un accès complet au plan de données des blobs, fichiers, tables et files d'attente."
keywords:
  - clé de compte de stockage
  - listKeys
  - plan de données
  - blob
  - accès complet
---

# Clés de stockage

Les deux **clés d'accès** d'un compte de stockage accordent un contrôle total et inconditionnel de son plan de données : chaque blob, partage de fichiers, table et file d'attente, indépendamment du RBAC. L'action de plan de gestion `Microsoft.Storage/storageAccounts/listKeys/action` (détenue par Contributor et de nombreux rôles personnalisés) les retourne, transformant un accès de gestion en accès total aux données.

## Lister et utiliser la clé

```bash
az storage account keys list --account-name <acct> --query '[0].value' -o tsv

KEY=$(az storage account keys list --account-name <acct> --query '[0].value' -o tsv)
az storage blob list --account-name <acct> --account-key "$KEY" --container-name <c>
az storage container list --account-name <acct> --account-key "$KEY"
```

Avec la clé, vous pouvez aussi générer un jeton SAS pour un accès durable et limité qui survit à la session d'un détenteur de clé :

```bash
az storage account generate-sas --account-name <acct> --account-key "$KEY" \
  --services bfqt --resource-types sco --permissions rwdlacup \
  --expiry 2030-01-01 -o tsv
```

## Notes d'exploitation

- La clé contourne entièrement Azure RBAC : un principal privé des rôles de données blob mais autorisé à `listKeys` lit et écrit tout de même toutes les données.
- Un SAS généré par soi-même est une persistance durable : il continue de fonctionner jusqu'à la rotation de la clé du compte, et la rotation est rare.
- Les clés déverrouillent aussi les partages Azure Files susceptibles d'être montés par des VM, une voie latérale vers les hôtes.

## Outils

- **az cli** (`storage account keys list`, `storage blob`).
- **MicroBurst** (`Get-AzPasswords`) : collecte les clés de stockage avec les autres secrets.
- **Azure Storage Explorer** : parcourir avec la clé récupérée.

## Références

- [Microsoft : gérer les clés de compte de stockage](https://learn.microsoft.com/azure/storage/common/storage-account-keys-manage)
- [HackTricks Cloud : stockage Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
