---
title: "SAS tokens: abusing over-scoped shared access signatures"
description: "Abusing Azure shared access signatures: over-scoped account, service, and user-delegation SAS URLs for durable data access."
keywords:
  - SAS token
  - shared access signature
  - user delegation
  - account SAS
  - data access
---

# SAS tokens

A shared access signature (SAS) is a signed query string that grants access to storage data without a login. There are three kinds: an **account SAS** (signed by the account key, can span services and operations), a **service SAS** (signed by the account key, scoped to one resource), and a **user-delegation SAS** (signed by an Entra token, scoped to the signer's RBAC). SAS URLs are routinely over-scoped (full permissions, year-long expiry) and leak into code, config, tickets, and logs, where they become durable, credential-free access.

## Minting a SAS from a key you hold

```bash
# account SAS: all services, all resource types, broad permissions, long expiry
az storage account generate-sas --account-name acme --account-key <KEY> \
  --services bfqt --resource-types sco \
  --permissions rwdlacup --expiry 2027-01-01 -o tsv

# container service SAS
az storage container generate-sas --account-name acme --account-key <KEY> \
  -n backups --permissions rl --expiry 2027-01-01 -o tsv
```

## Using a found SAS

```bash
# the SAS is the whole credential; append it to the resource URL
curl "https://acme.blob.core.windows.net/backups/db.bak?<SAS>"
az storage blob list --account-name acme --container-name backups --sas-token '<SAS>'
```

## Exploitation notes

- A SAS cannot be revoked individually unless it is tied to a stored access policy; an account-key-signed SAS stays valid until expiry or a key rotation, so a leaked long-lived SAS is durable access and a persistence foothold.
- Decode the permission and expiry fields of any SAS you find (`sp=`, `se=`, `sr=`, `ss=`, `srt=`): `sp=rwdlacup` with a distant `se=` is full control.
- A user-delegation SAS is only as scoped as the signer's RBAC, but it survives the signer's session; an account SAS ignores RBAC entirely.
- Grep repos, pipeline variables, and ARM template outputs for `?sv=...&sig=` SAS strings.

## Tools

- **az CLI** (`storage ... generate-sas`): mint and use SAS.
- **MicroBurst**: surfaces keys and SAS during storage enumeration.
- **TruffleHog** / secret scanners: find leaked SAS URLs in source and config.

## References

- [HackTricks Cloud: Azure SAS tokens](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [Microsoft: shared access signatures](https://learn.microsoft.com/azure/storage/common/storage-sas-overview)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
