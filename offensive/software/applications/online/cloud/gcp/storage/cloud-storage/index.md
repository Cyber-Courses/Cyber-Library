---
title: "Cloud Storage"
description: "Attacking Cloud Storage buckets: enumerating public and misconfigured buckets and abusing bucket IAM and ACLs to read or write objects."
keywords:
  - Cloud Storage
  - GCS
  - bucket
  - IAM
  - ACL
---

# Cloud Storage

Cloud Storage (GCS) buckets share a single global namespace, so they are discoverable without a credential, and access is decided by either **bucket IAM** (uniform) or **legacy ACLs** (fine-grained). The two failure modes are a bucket exposed to `allUsers`/`allAuthenticatedUsers` and an over-broad IAM binding (`roles/storage.admin` or `objectViewer` on the project), both of which hand an attacker the objects inside.

## What folds in here

- **[Enumeration](enumeration.md)**: finding buckets and readable objects by name guessing and listing.
- **[Access](access.md)**: reading or writing objects through broad bucket IAM, ACLs, or anonymous bindings.

## References

- [HackTricks Cloud: GCP buckets](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [GCPBucketBrute (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/GCPBucketBrute)
- [Google: Cloud Storage access control](https://cloud.google.com/storage/docs/access-control)
