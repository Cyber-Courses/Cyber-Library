---
title: "AWS data"
description: "Reaching data in AWS managed data stores: RDS databases and their snapshots, DynamoDB tables, and the queues and analytics stores, through API access, snapshot sharing, and over-broad resource policies."
keywords:
  - RDS
  - DynamoDB
  - snapshot
  - resource policy
  - data stores
---

# Data

Beyond object storage, AWS holds data in managed **databases** and stores: RDS, DynamoDB, Redshift, and the messaging and analytics services. These are reached through the API with the right permissions, and often through **snapshots** and over-broad **resource policies** rather than the database network port.

The pages here cover dumping **DynamoDB** tables through the API, recovering **RDS** data by **sharing or restoring a snapshot** into a controlled account (the managed-service analogue of stealing an EBS snapshot), and spotting over-broad resource policies that expose a store cross-account.

## What folds in here

- **Exfiltration** from managed data stores.
- Network-level database attacks (authenticating to the engine over its port) live in the [Database](../../../database/index.md) area; this surface is about the AWS control-plane paths to the same data.

## References

- [HackTricks Cloud: AWS RDS and DynamoDB](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: sharing a DB snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html)
