---
title: "Enumeration: finding public containers and storage accounts"
order: 1
description: "Discovering public Azure blob containers and storage accounts through naming guesses and anonymous listing."
keywords:
  - blob enumeration
  - public container
  - storage account
  - anonymous
  - discovery
---

# Enumeration

Storage account names are global and DNS-resolvable (`<name>.blob.core.windows.net`), so the account namespace is brute-forceable from outside with no credentials, and any container left at a public access level lists and serves its blobs anonymously. Enumeration is guessing account and container names, then confirming anonymous listing.

## Finding storage accounts

```bash
# resolve candidate account names (org-themed wordlists), valid = DNS answer
for n in acme acmeprod acmebackups acmedata; do
  host $n.blob.core.windows.net | grep -q address && echo "exists: $n"
done
```

MicroBurst automates the subdomain and container sweep across the storage endpoints (blob, file, queue, table):

```
Invoke-EnumerateAzureSubDomains -Base acme -Verbose
Invoke-EnumerateAzureBlobs -Base acme          # guesses containers and lists public ones
```

## Confirming a public container

```bash
# anonymous container listing over REST (no auth)
curl "https://acme.blob.core.windows.net/backups?restype=container&comp=list"

# with the CLI, no credential
az storage blob list --account-name acme --container-name backups --auth-mode login
```

A `200` with a blob list (or an XML `EnumerationResults`) means the container is public; a `PublicAccessNotPermitted` means it exists but is locked.

## Exploitation notes

- Public access is set per container; an account can have one world-readable container beside locked ones, so enumerate container names even when the account default is private.
- `goblob`, `basacwler`, and `cloud_enum` run the same brute at scale against common container names (backup, logs, config, web, public).
- Resolve candidate names before listing: a DNS answer confirms the account exists and avoids noisy listing attempts against nonexistent names.

## Tools

- **MicroBurst** (`Invoke-EnumerateAzureBlobs`, `Invoke-EnumerateAzureSubDomains`): account and container discovery.
- **cloud_enum**: multi-cloud public-storage brute including Azure blobs.
- **az CLI** (`storage blob list --auth-mode login`): anonymous and authenticated listing.

## References

- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud: Azure storage enumeration](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [cloud_enum](https://github.com/initstring/cloud_enum)
- [Microsoft: Blob service REST list containers](https://learn.microsoft.com/rest/api/storageservices/list-containers2)
