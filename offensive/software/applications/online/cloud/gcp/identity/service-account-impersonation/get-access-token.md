---
title: "getAccessToken: mint an OAuth token for a service account"
description: "Minting a short-lived OAuth access token for a target service account with iam.serviceAccounts.getAccessToken (generateAccessToken)."
keywords:
  - getAccessToken
  - generateAccessToken
  - access token
  - service account
  - impersonation
---

# getAccessToken

`iam.serviceAccounts.getAccessToken` (the `generateAccessToken` API, held by `roles/iam.serviceAccountTokenCreator`) mints a short-lived OAuth access token for a target service account. It is the most direct impersonation: no key, no deploy, just ask IAM Credentials for the target's token and act as it immediately.

## Mint and use the token

```bash
# built into gcloud: run any command as the target SA
gcloud compute instances list --impersonate-service-account <target>@<proj>.iam.gserviceaccount.com

# or mint the raw token
gcloud auth print-access-token --impersonate-service-account <target>@<proj>.iam.gserviceaccount.com

# REST
curl -s -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/<target>@<proj>.iam.gserviceaccount.com:generateAccessToken" \
  -d '{"scope":["https://www.googleapis.com/auth/cloud-platform"]}'
```

## Exploitation notes

- Token Creator on a more privileged SA is a complete escalation; enumerate who you can impersonate from the IAM policy.
- The token is short-lived (default one hour) but re-mintable as long as you hold the permission.
- Chain it: impersonate SA-A, then use A's Token Creator on SA-B to reach B ([implicit delegation](implicit-delegation.md)).

## Tools

- **gcloud** (`--impersonate-service-account`, `auth print-access-token`).

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [HackTricks Cloud: service account impersonation](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google: create short-lived credentials](https://cloud.google.com/iam/docs/create-short-lived-credentials-direct)
