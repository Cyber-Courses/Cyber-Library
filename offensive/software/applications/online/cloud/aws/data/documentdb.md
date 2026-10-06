---
title: "DocumentDB: reaching Mongo-compatible clusters through exposure"
order: 6
description: "Reaching DocumentDB clusters through exposed endpoints and weak credentials to read Mongo-compatible data."
keywords:
  - DocumentDB
  - MongoDB
  - cluster
  - endpoint
  - database
---

# DocumentDB

DocumentDB is AWS's MongoDB-compatible database. It has no IAM data plane: access is the cluster endpoint plus a database username and password, so it falls to the same exposure pattern as self-hosted Mongo. Credentials typically live in [Secrets Manager](../credentials/secret-stores/secrets-manager.md) or an application's config, and a cluster reachable from a compromised VPC host is one credential away from a full read.

## Finding and connecting

```bash
aws docdb describe-db-clusters \
  --query 'DBClusters[].[DBClusterIdentifier,Endpoint,Port]'
# connect with the mongo shell (TLS on by default)
mongosh "mongodb://<user>:<pw>@<endpoint>:27017/?tls=true&tlsCAFile=global-bundle.pem"
```

## Exploitation notes

- DocumentDB is almost always inside a VPC, so this is a post-foothold technique reached from an EC2 box or through a pivot, not from the internet.
- Credentials recovered from Secrets Manager or an app server are the usual key; the database itself has no second factor.
- Dump collections with `mongodump` once connected for offline analysis.

## Tools

- **AWS CLI** (`docdb describe-db-clusters`).
- **mongosh / mongodump**: native MongoDB clients.

## References

- [AWS: connecting to a DocumentDB cluster](https://docs.aws.amazon.com/documentdb/latest/developerguide/connect.html)
- [HackTricks Cloud: AWS DocumentDB](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable cloud data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
