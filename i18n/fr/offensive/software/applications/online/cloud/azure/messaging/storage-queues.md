---
title: "Storage Queues : lire et injecter des messages de queue"
order: 4
description: "Lire et écrire des Azure Storage Queues via des clés de compte ou SAS pour la capture et l'injection de messages."
keywords:
  - Storage Queues
  - clé de compte
  - SAS
  - message
  - queue
---

# Storage Queues

Les Storage Queues sont le service de file d'attente simple intégré à un compte de stockage Azure, utilisé pour tamponner le travail entre les couches applicatives. Comme elles résident dans le compte de stockage, toute **clé de compte** ou **SAS** qui atteint le compte (voir [clés de stockage](../credentials/storage-keys.md)) atteint aussi ses queues, ce qui permet de lire les messages en transit, d'injecter du travail forgé que le consommateur traitera, ou de vider une queue pour nier le service derrière elle.

## Énumérer et accéder

```bash
az storage queue list --account-name <acct> --account-key <key> -o table
az storage message peek --queue-name <q> --account-name <acct> --account-key <key> --num-messages 10
```

## Lire, injecter et vider

```bash
# read work without dequeuing (peek leaves it for the real consumer)
az storage message peek --queue-name <q> --account-name <acct> --account-key <key> --num-messages 32

# inject a forged message the consumer trusts
az storage message put --queue-name <q> --content '{"job":"forged"}' \
  --account-name <acct> --account-key <key>

# clear the queue to deny the downstream worker
az storage message clear --queue-name <q> --account-name <acct> --account-key <key>
```

## Notes d'exploitation

- `peek` ne retire pas le message ni ne fixe de délai de visibilité, donc la lecture est invisible pour le consommateur ; `get` masquerait le message pendant sa fenêtre de délai.
- Le Base64 est l'encodage par défaut qu'utilisent de nombreux SDK, donc décodez le contenu peeké avant de le lire et encodez les corps forgés pour correspondre à ce que le consommateur attend.
- Une clé de compte de stockage est souvent récupérée depuis les [app settings](../credentials/app-settings-and-connection-strings.md) ou une connection string, donc l'accès aux queues vient généralement gratuitement avec la compromission du stockage.

## Outils

- **Azure CLI** (`az storage message` / `az storage queue`).
- **Azure Storage Explorer** : lecture et injection via interface graphique.
- **MicroBurst** : fait ressortir les clés de compte de stockage et les connection strings depuis la configuration.

## Références

- [HackTricks Cloud : Azure Storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft : autorisation Queue Storage](https://learn.microsoft.com/azure/storage/queues/authorize-data-operations-cli)
