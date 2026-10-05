---
title: "Service account impersonation"
description: "Minting credentials for a target service account: getAccessToken, signJwt and signBlob, Token Creator grants, implicit delegation, and long-lived key creation."
keywords:
  - service account impersonation
  - getAccessToken
  - signJwt
  - Token Creator
  - delegation
  - key creation
---

# Service account impersonation

Impersonation is the most direct GCP escalation: a permission on a target **service account** lets you act as it without deploying anything. The `roles/iam.serviceAccountTokenCreator` role (and the individual permissions below) is the hinge, and it is frequently granted too broadly.

## The primitives

- **[getAccessToken](get-access-token.md)**: mint a short-lived OAuth token for the target directly.
- **[signJwt and signBlob](sign-jwt-and-sign-blob.md)**: forge a signed JWT or blob as the target, then exchange it for a token.
- **[Token Creator grant](token-creator-grant.md)**: grant yourself Token Creator on the target with `serviceAccounts.setIamPolicy`, then impersonate.
- **[Implicit delegation](implicit-delegation.md)**: chain through an intermediate account to reach one you cannot call directly.
- **[Key creation](key-creation.md)**: create a long-lived JSON key for durable access and persistence.

## Using an impersonated identity

Most `gcloud` commands accept the impersonation flag once you hold the permission:

```bash
gcloud <command> --impersonate-service-account=<target-sa>@<project>.iam.gserviceaccount.com
gcloud auth print-access-token --impersonate-service-account=<target-sa>@...
```

## References

- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP service account impersonation](https://hackingthe.cloud/)
- [Google: service account impersonation](https://cloud.google.com/iam/docs/service-account-impersonation)
