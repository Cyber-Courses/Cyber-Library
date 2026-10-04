---
title: "RDS"
description: "Attacking RDS and Aurora: exposing data through shared or public snapshots and reaching the database through weak network and auth controls."
keywords:
  - RDS
  - Aurora
  - snapshot
  - database
  - IAM auth
---

# RDS

RDS and Aurora hold relational data behind two separable surfaces. The **control plane** lets a principal with RDS permissions copy or share a **snapshot**, restore it into an instance they own, and read the whole database offline without ever touching the original. The **data plane** is the running instance, reachable when its endpoint is public, its security group is loose, or its credentials are weak or recoverable.

## What folds in here

- **[Snapshots](snapshots.md)**: sharing, copying, and restoring snapshots to read the database offline.
- **[Access](access.md)**: reaching live instances through exposed endpoints, weak credentials, or RDS IAM authentication.

## References

- [HackTricks Cloud: AWS RDS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: sharing a DB snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html)
- [Pacu: AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
