---
title: "AWS storage"
description: "Reaching data in AWS storage: S3 bucket and object exposure through public access, ACLs, and bucket policies, bucket enumeration and takeover, and EBS snapshot exposure as a route to disk contents and secrets."
keywords:
  - S3
  - bucket policy
  - public access
  - EBS snapshot
  - exfiltration
---

# Storage

S3 is the most common cloud data-exposure surface: buckets made public, over-broad **bucket policies** and ACLs, and predictable names that enumerate. Storage is also where **exfiltration** lands, so data theft is documented here rather than as a separate phase.

The pages here cover finding and reading public and misconfigured **S3** buckets, enumerating bucket names and objects, **bucket takeover** of dangling references, and recovering disk contents and secrets from exposed or shared **EBS snapshots**.

## What folds in here

- **Exfiltration** of account data (objects, snapshots) is treated as a storage outcome.
- Snapshot theft overlaps with [compute](../compute/index.md); the snapshot-sharing mechanics live there, the data-recovery angle here.

## References

- [HackTricks Cloud: AWS S3](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: S3 bucket policies and access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-policy-language-overview.html)
