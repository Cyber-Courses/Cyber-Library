---
title: "HMAC keys: minting a Cloud Storage HMAC key for a service account"
description: "Creating a Cloud Storage HMAC key for a service account with storage.hmacKeys.create to access buckets as that account."
keywords:
  - HMAC keys
  - storage.hmacKeys.create
  - Cloud Storage
  - interoperability
  - service account
  - S3 API
---

# HMAC keys

Cloud Storage supports S3-compatible **HMAC keys** for interoperability. `storage.hmacKeys.create` lets you mint an access-key/secret pair bound to a service account, and that pair then reaches every bucket the account can, through the XML (S3) API, outside the usual OAuth path and its logging. It is both a lateral step (act as the service account against storage) and a durable, low-visibility credential.

## Creating and using a key

```bash
# mint an HMAC key for a service account you can act for
gsutil hmac create <sa>@<project>.iam.gserviceaccount.com
# -> Access ID and Secret

# use it with any S3 client against the GCS XML endpoint
AWS_ACCESS_KEY_ID=<accessid> AWS_SECRET_ACCESS_KEY=<secret> \
  aws --endpoint-url https://storage.googleapis.com s3 ls
```

## Exploitation notes

- The key inherits the service account's storage access, so target an account with broad bucket roles.
- HMAC access runs through the S3-style API, which some monitoring and VPC Service Controls handle differently from native GCS calls, making it a quieter path.
- Keys persist until explicitly deleted, independent of the account's tokens, so this doubles as persistence.

## Tools

- **gsutil** (`hmac create`, `hmac list`).
- Any **S3 client** (`aws s3`, boto3) pointed at `storage.googleapis.com`.

## References

- [Google: managing HMAC keys](https://cloud.google.com/storage/docs/authentication/managing-hmackeys)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
