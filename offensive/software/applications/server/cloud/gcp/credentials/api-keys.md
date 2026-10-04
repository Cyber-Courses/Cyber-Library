---
title: "API keys: creating and listing Google API keys"
description: "Creating or listing Google API keys with serviceusage.apiKeys.create and list for durable, unscoped API access."
keywords:
  - API keys
  - serviceusage.apiKeys
  - apiKeys.create
  - unscoped
  - persistence
  - Google API
---

# API keys

Google **API keys** are standalone strings that authenticate calls to enabled APIs without an identity or token expiry. `serviceusage.apiKeys.create` mints one and `serviceusage.apiKeys.list` (with `getKeyString`) recovers existing ones, giving durable access to whatever APIs the key is allowed, frequently with no restriction set at all.

## Listing and recovering existing keys

```bash
gcloud services api-keys list --format='value(name,displayName)'
gcloud services api-keys get-key-string <key-id>
```

## Creating a new key

```bash
gcloud services api-keys create --display-name="legit-looking"
# returns the key string; unrestricted unless --api-target / --allowed-* is set
```

## Exploitation notes

- Unrestricted keys call any enabled API in the project, and because they are not tied to an identity they sidestep IAM-token revocation, which makes a created key quiet persistence.
- Existing keys leak into mobile apps, front-end JavaScript, and configs; recovering the key string turns a code leak into live project access.
- Keys do not appear in IAM policy, so defenders auditing bindings miss them.

## Tools

- **gcloud** (`services api-keys create / list / get-key-string`).

## References

- [Google: API keys](https://cloud.google.com/docs/authentication/api-keys)
- [HackTricks Cloud: GCP API keys](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
