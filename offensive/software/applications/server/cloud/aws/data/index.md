---
title: "AWS data"
description: "Attacking AWS data services: RDS snapshots and access, DynamoDB, Redshift, SageMaker, Glue, Athena, Lake Formation, DocumentDB, ElastiCache, EMR, Neptune, Timestream, and Keyspaces."
keywords:
  - RDS
  - DynamoDB
  - Redshift
  - SageMaker
  - Glue
  - Athena
  - Lake Formation
---

# Data

The managed data stores are where the engagement pays off: the database, the warehouse, the lake, and the cache hold what the account exists to protect. Reaching them splits into two moves that recur across every service here. Either the **data plane** is exposed directly (a publicly reachable endpoint, weak or shared credentials, an over-broad grant), or the **control plane** hands you the data offline (a shared snapshot, an export to S3, a credential-vending call). Several of these services also run jobs under an attached role, so they double as compute and are cross-referenced into [identity](../identity/index.md) where that role is the prize.

## What folds in here

- **[RDS](rds/index.md)**: snapshot sharing and restore, and reaching the instance through weak network and auth controls.
- **[DynamoDB](dynamodb.md)**: `Scan`, `Query`, and table export to S3.
- **[SageMaker](sagemaker.md)**: notebooks, training jobs, and endpoints, and the attached role.
- **[Redshift](redshift.md)**: exposed clusters, temporary credentials, and database grants.
- **[Glue](glue.md)**: the Data Catalog, job scripts, and running jobs under the Glue role.
- **[DocumentDB](documentdb.md)**: exposed Mongo-compatible clusters.
- **[ElastiCache](elasticache.md)**: unauthenticated Redis and Memcached.
- **[Athena](athena.md)**: querying S3 data through the catalog.
- **[Lake Formation](lake-formation.md)**: governed-table permissions and credential vending.
- **[EMR](emr.md)**: clusters, steps, and the instance role.
- **[Neptune](neptune.md)**: exposed graph-database clusters.
- **[Timestream](timestream.md)**: querying time-series databases.
- **[Keyspaces](keyspaces.md)**: Cassandra-compatible tables.

## References

- [HackTricks Cloud: AWS databases](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: security in Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.html)
- [CloudFox: enumerating exploitable cloud data services](https://github.com/BishopFox/cloudfox)
- [Prowler: AWS data-exposure and misconfiguration checks](https://github.com/prowler-cloud/prowler)
