---
title: "GCP credentials"
description: "Harvesting GCP credentials: the instance metadata server, Secret Manager, service-account keys, cached gcloud and ADC tokens, Cloud KMS, HMAC keys, and API keys."
keywords:
  - GCP credentials
  - metadata
  - Secret Manager
  - service account keys
  - access tokens
  - KMS
---

# Credentials

Every GCP API call carries a credential, and the fastest way to widen access is to harvest more of them. Credentials arrive as short-lived **OAuth access tokens** (from the metadata server or `gcloud`), long-lived **service-account JSON keys**, and the secrets and keys that services hand back when read. Each has different theft and reuse characteristics, and several services mint fresh credentials for whoever can call them.

## What folds in here

- **[Instance metadata](instance-metadata/index.md)**: the attached service account's token and SSH keys from `metadata.google.internal`, including through SSRF.
- **[Secret Manager](secret-manager.md)**: reading stored secrets with `secretmanager.versions.access`.
- **[Service account keys](service-account-keys.md)**: exported JSON keys, and creating new ones for durable access.
- **[Access tokens](access-tokens.md)**: looting cached `gcloud` and Application Default Credentials from disk.
- **[Cloud KMS](cloud-kms.md)**: decrypting data and signing as a key.
- **[HMAC keys](hmac-keys.md)**: minting a Cloud Storage HMAC key for a service account.
- **[API keys](api-keys.md)**: creating or listing Google API keys for durable, unscoped access.

## References

- [HackTricks Cloud: GCP credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: service account credentials](https://cloud.google.com/iam/docs/service-account-creds)
