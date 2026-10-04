---
title: "Cloud SQL: reaching databases through the proxy, public IP, and built-in users"
description: "Attacking Cloud SQL: database access through the SQL Auth Proxy, public IP and authorized networks, and the built-in database users."
keywords:
  - Cloud SQL
  - SQL Auth Proxy
  - authorized networks
  - database
  - public IP
  - MySQL
---

# Cloud SQL

Cloud SQL runs managed MySQL, PostgreSQL, and SQL Server. Reaching the data is either a network problem (a public IP with a wide authorized-network range, or the SQL Auth Proxy opened from a host you hold) or a credential problem (built-in DB users whose passwords sit in metadata, Secret Manager, or app config, and IAM database authentication where your principal is granted a DB role).

## Finding and reaching instances

```bash
gcloud sql instances list
gcloud sql instances describe <inst> \
  --format='value(ipAddresses,settings.ipConfiguration.authorizedNetworks)'
# a public IP + 0.0.0.0/0 authorized network is directly reachable
```

## Connecting

```bash
# built-in user over a reachable IP
mysql -h <public-ip> -u root -p

# or tunnel through the SQL Auth Proxy with your gcloud creds
cloud-sql-proxy <project>:<region>:<inst> &
psql "host=127.0.0.1 dbname=postgres user=postgres"

# IAM database authentication: mint a token as your principal
gcloud sql generate-login-token
```

## Exploitation notes

- `cloudsql.instances.connect` plus the proxy reaches a private-IP instance without a password when IAM DB auth is enabled for your principal.
- Built-in user passwords are frequently reused from [Secret Manager](../credentials/secret-manager.md) or an instance's [metadata](../credentials/instance-metadata/index.md); harvest there first.
- A database export (`gcloud sql export sql`) to a bucket you control dumps the whole database offline when you hold `cloudsql.instances.export`.

## Tools

- **gcloud** (`sql instances list/describe`, `sql export sql`).
- **cloud-sql-proxy**: authenticated tunnel to an instance.
- **native clients** (`mysql`, `psql`, `sqlcmd`).

## References

- [HackTricks Cloud: GCP Cloud SQL](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Cloud SQL Auth Proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy)
