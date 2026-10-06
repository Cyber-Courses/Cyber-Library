---
title: "Bigtable: reading tables through bigtable.tables access"
order: 8
description: "Attacking Cloud Bigtable: reading tables through bigtable.tables access with broad instance roles."
keywords:
  - Bigtable
  - table
  - bigtable.tables
  - instance
  - NoSQL
  - data access
---

# Bigtable

Cloud Bigtable is a wide-column NoSQL store reached through IAM. A principal with `bigtable.tables.readRows` (or a broad instance role like `roles/bigtable.reader`) reads any table in the instance, so the attack is enumerating reachable instances and tables and dumping their rows.

## Reading a table

```bash
gcloud bigtable instances list
cbt -instance <inst> ls                 # list tables
cbt -instance <inst> read <table> count=1000
```

## Exploitation notes

- Instance-level roles apply to every table; a single broad binding exposes the whole instance.
- Write roles (`bigtable.tables.mutateRows`) allow row tampering where the data backs application state.
- Bigtable has no public endpoint; access is the IAM binding on your principal or an impersonated service account.

## Tools

- **cbt** / **gcloud** (`bigtable instances list`, `cbt read`).
- **Bigtable client libraries** for scripted scans.

## References

- [HackTricks Cloud: GCP Bigtable](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Bigtable access control](https://cloud.google.com/bigtable/docs/access-control)
