---
title: "Key creation: mint a long-lived key for a service account"
description: "Creating a long-lived JSON key for a target service account with iam.serviceAccountKeys.create for durable access and persistence."
keywords:
  - serviceAccountKeys.create
  - service account key
  - JSON key
  - persistence
  - impersonation
---

# Key creation

`iam.serviceAccountKeys.create` mints a user-managed JSON key for a target service account. Unlike a short-lived impersonation token, the key is long-lived and portable: it is both an immediate takeover of the account and durable persistence that survives password and session changes.

## Create and activate a key

```bash
gcloud iam service-accounts keys create key.json \
  --iam-account <target>@<proj>.iam.gserviceaccount.com

# authenticate as the target from anywhere
gcloud auth activate-service-account --key-file key.json
gcloud auth print-access-token
```

## Exploitation notes

- The key never expires by default, so it is a favourite persistence mechanism: exfiltrate `key.json` and return at will.
- Key creation on a privileged SA is a full escalation; enumerate which accounts you can create keys for.
- Noisier than [getAccessToken](get-access-token.md) and [signJwt](sign-jwt-and-sign-blob.md) because it leaves a key object listed on the account; prefer those when you only need transient access.

## Tools

- **gcloud** (`iam service-accounts keys create`).

## References

- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: managing service account keys](https://cloud.google.com/iam/docs/keys-create-delete)
