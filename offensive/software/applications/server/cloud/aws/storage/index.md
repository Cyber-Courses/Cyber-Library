---
title: "AWS storage"
description: "Attacking AWS storage: S3 bucket enumeration, public access and bucket-policy and ACL abuse, EBS and EFS data recovery, FSx and Storage Gateway access, and AWS Backup vaults."
keywords:
  - S3
  - EBS
  - EFS
  - bucket policy
  - snapshots
---

# Storage

Storage is where the data lives, so it is the usual objective once a principal is held. AWS storage breaks into object storage (S3), block storage (EBS, exposed through its snapshots), shared file systems (EFS, FSx), the backup plane (AWS Backup), and the hybrid bridge (Storage Gateway). Most storage compromise is not an exploit but a permission or exposure problem: a world-readable bucket, a snapshot shared to all accounts, a mount target reachable from a subnet you control.

## What folds in here

- **[S3](s3/index.md)**: bucket and object discovery, public access, and bucket-policy and ACL abuse.
- **[EBS snapshots](ebs-snapshots.md)**: public or shared block-storage snapshots restored to read their volumes.
- **[EFS](efs.md)**: exposed NFS file systems mounted through permissive security groups or file-system policy.
- **[Backup](backup.md)**: recovery points read and restored out of AWS Backup vaults.
- **[FSx](fsx.md)**: Windows, Lustre, and NetApp file systems reached through share permissions.
- **[Storage Gateway](storage-gateway.md)**: file shares and cached volumes bridging on-premises access to S3 and EBS.

Enumeration is folded into each page: finding the resource is the first half of reaching its data.

## References

- [HackTricks Cloud: AWS S3](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [Rhino Security Labs: penetration testing AWS storage](https://rhinosecuritylabs.com/aws/penetration-testing-aws-storage/)
