---
title: "Tables de stockage : lire Table storage avec des clés ou un SAS"
order: 5
description: "Lire Azure Table storage via les clés de compte ou un SAS pour obtenir des données structurées."
keywords:
  - Tables de stockage
  - Table storage
  - clé de compte
  - SAS
  - NoSQL
---

# Tables de stockage

Azure Table storage est le service NoSQL clé-valeur au sein d'un compte de stockage, et il partage l'authentification du compte : la **clé de compte** ou un jeton **SAS** lit et écrit chaque table. Les applications l'utilisent pour l'état de session, la configuration et les données d'audit, de sorte qu'une clé de stockage récupérée expose souvent des tables contenant des jetons, des enregistrements d'utilisateurs ou un état interne.

## Lister et lire les tables

```bash
KEY=$(az storage account keys list -n <acct> -g <rg> --query '[0].value' -o tsv)
az storage table list --account-name <acct> --account-key "$KEY" -o table
az storage entity query --account-name <acct> --account-key "$KEY" --table-name <t>
```

## Notes d'exploitation

- L'accès aux tables repose sur la même clé de compte que le [stockage blob](../storage/blob-storage/index.md), de sorte qu'un seul `listKeys` accorde les deux ; vérifiez ce que la clé déverrouille déjà avant toute action plus bruyante.
- Un jeton SAS limité au service de tables atteint les tables même sans la clé de compte, et de tels jetons fuient dans la configuration des applications et les URL.
- Les tables qui adossent l'autorisation applicative (lignes de rôles, feature flags) font d'une écriture une escalade au niveau de l'application.

## Outils

- **az cli** (`az storage table list`, `az storage entity query`).
- **Azure Storage Explorer** : parcourir les tables avec une clé ou un SAS.

## Références

- [Microsoft : autorisation Table storage](https://learn.microsoft.com/azure/storage/tables/authorize-access-azure-active-directory)
- [HackTricks Cloud : stockage Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
