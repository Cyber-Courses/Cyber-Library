---
title: "S3"
order: 1
description: "Attacking S3: discovering buckets and objects, exploiting public access and ACLs, and abusing writable or readable bucket policies."
keywords:
  - S3
  - bucket
  - public access
  - bucket policy
  - ACL
---

# S3

S3 holds objects in globally named **buckets**, and the global namespace is the first weakness: a bucket name is guessable, resolves in DNS, and tells you whether it exists before you hold any credential. Access is decided by three overlapping controls (the account Block Public Access setting, the bucket policy, and object and bucket ACLs), and a misconfiguration in any one exposes data or grants write.

## What folds in here

- **[Enumeration](enumeration.md)**: finding buckets and objects by naming, DNS, and permission probing.
- **[Public access](public-access.md)**: buckets left world-readable or world-writable by ACLs or disabled Block Public Access.
- **[Bucket policy and ACL](bucket-policy-and-acl.md)**: over-broad policies and ACLs that grant cross-account read, write, or takeover.

A writable bucket that backs a website or a software distribution turns into code execution downstream, and a bucket name freed by a deleted resource is reclaimable, covered under [dangling-DNS takeover](../../networking/dangling-dns-takeover.md).

## References

- [HackTricks Cloud: S3 enumeration](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [PayloadsAllTheThings: AWS S3](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cloud%20-%20AWS%20Pentest.md)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [GrayhatWarfare: public buckets](https://buckets.grayhatwarfare.com/)
- [S3Scanner](https://github.com/sa7mon/S3Scanner)
- [cloud_enum](https://github.com/initstring/cloud_enum)
