---
title: "Keyspaces: reaching Cassandra-compatible tables"
description: "Accessing Keyspaces (Cassandra-compatible) tables through exposed credentials and over-broad grants."
keywords:
  - Keyspaces
  - Cassandra
  - CQL
  - table
  - credentials
---

# Keyspaces

Keyspaces is AWS's Cassandra-compatible database. It is reached either with service-specific credentials (a generated username and password tied to an IAM user) or with SigV4 signing, and a principal holding `cassandra:Select` reads any table the policy allows. Over-broad grants and recoverable service-specific credentials are the way in.

## Connecting and reading

```bash
aws keyspaces list-keyspaces ; aws keyspaces list-tables --keyspace-name <ks>

# cqlsh with service-specific credentials over TLS (port 9142)
cqlsh cassandra.<region>.amazonaws.com 9142 --ssl \
  -u <svc-user> -p <svc-pass>
> SELECT * FROM ks.table LIMIT 100;
```

## Exploitation notes

- Service-specific credentials are generated per IAM user and, once recovered, give direct CQL access independent of further IAM checks.
- A SigV4 plugin for the Cassandra driver lets a holding principal connect without a stored password at all.
- Broad `cassandra:Select` at the service level reads every table in the account's keyspaces.

## Tools

- **AWS CLI** (`keyspaces list-keyspaces`, `list-tables`).
- **cqlsh** with the SigV4 auth plugin: native CQL access.

## References

- [AWS: Keyspaces access and credentials](https://docs.aws.amazon.com/keyspaces/latest/devguide/security_iam_service-with-iam.html)
- [HackTricks Cloud: AWS Keyspaces](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
