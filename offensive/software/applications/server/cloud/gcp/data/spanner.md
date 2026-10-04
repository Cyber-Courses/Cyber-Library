---
title: "Spanner: reading databases through spanner.databases access"
description: "Attacking Cloud Spanner: reading databases through spanner.databases access and session abuse."
keywords:
  - Cloud Spanner
  - database
  - spanner.databases
  - session
  - SQL
  - data access
---

# Spanner

Cloud Spanner is a globally distributed relational database reached through IAM. A principal holding `spanner.databases.read` (or `roles/spanner.databaseReader`) opens a session and queries any table in the database, so the attack is simply enumerating the instances and databases a token can reach and reading them.

## Reading a database

```bash
gcloud spanner instances list
gcloud spanner databases list --instance <inst>
gcloud spanner databases execute-sql <db> --instance <inst> \
  --sql='SELECT * FROM Users LIMIT 1000'
```

## Exploitation notes

- `spanner.databases.beginOrRollbackReadWriteTransaction` and related write permissions let you tamper with rows where the role is broad.
- Spanner has no public network surface, so access is purely the IAM binding on your principal or an impersonated service account; chase the binding rather than the network.

## Tools

- **gcloud** (`spanner databases execute-sql`).
- **Spanner client libraries** for scripted session reads.

## References

- [HackTricks Cloud: GCP Spanner](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Spanner IAM](https://cloud.google.com/spanner/docs/iam)
