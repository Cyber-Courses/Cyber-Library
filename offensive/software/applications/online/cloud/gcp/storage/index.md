---
title: "GCP storage"
description: "Attacking GCP storage: Cloud Storage bucket enumeration and IAM and ACL access, persistent-disk snapshots, and Filestore shares."
keywords:
  - Cloud Storage
  - bucket
  - disk snapshot
  - Filestore
  - IAM
---

# Storage

GCP keeps bulk data in Cloud Storage buckets, in the persistent disks behind Compute Engine, and in Filestore NFS shares. Each is reached differently: buckets by a global name plus IAM or legacy ACLs, disks by snapshotting and re-attaching them, and Filestore by mounting the share from inside the VPC. The work is finding the store you can reach and pulling the credentials, keys, and databases inside.

## What folds in here

- **[Cloud Storage](cloud-storage/index.md)**: enumerating public and misconfigured buckets and abusing bucket IAM and ACLs.
- **[Disk snapshots](disk-snapshots.md)**: copying a persistent disk and re-attaching it to read another instance's contents.
- **[Filestore](filestore.md)**: mounting NFS shares exposed within the VPC.

HMAC keys that grant S3-interop access to Cloud Storage are a credential and live under [credentials](../credentials/hmac-keys.md).

## References

- [HackTricks Cloud: GCP storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [GCPBucketBrute (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/GCPBucketBrute)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
