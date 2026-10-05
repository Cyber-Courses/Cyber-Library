---
title: "setIamPolicy: grant yourself any role at a scope"
description: "Granting yourself any role by calling resourcemanager.projects, folders, or organizations setIamPolicy on a resource you can administer."
keywords:
  - setIamPolicy
  - IAM binding
  - role grant
  - resourcemanager
  - owner
---

# setIamPolicy

`setIamPolicy` is the ability to rewrite a resource's IAM policy. If you hold `resourcemanager.projects.setIamPolicy` (or the folder or organization variant, or a resource-level `*.setIamPolicy`), you add a binding granting yourself `roles/owner` and the escalation is complete in one call. Many predefined roles carry a `setIamPolicy` permission for their service without it being obvious.

## Grant yourself Owner on the project

```bash
gcloud projects add-iam-policy-binding <project> \
  --member="user:you@example.com" --role="roles/owner"
# or for a controlled service account
gcloud projects add-iam-policy-binding <project> \
  --member="serviceAccount:<sa>@<project>.iam.gserviceaccount.com" --role="roles/owner"
```

At folder or organization scope the same move grants control of every project beneath:

```bash
gcloud resource-manager folders add-iam-policy-binding <folder-id> --member=... --role="roles/owner"
gcloud organizations add-iam-policy-binding <org-id> --member=... --role="roles/resourcemanager.organizationAdmin"
```

## Resource-scoped variants

Where you only hold `setIamPolicy` on one resource type, grant yourself a role on that resource and pivot from it (for example `iam.serviceAccounts.setIamPolicy` on a service account grants Token Creator, covered under [Token Creator grant](../service-account-impersonation/token-creator-grant.md)).

## Exploitation notes

- `add-iam-policy-binding` reads then writes the policy; a race with a concurrent change can drop your binding, so re-read to confirm it stuck.
- Owner at a lower scope is enough for most goals; grabbing org-level admin is louder and often unnecessary.
- The binding is visible in the resource's IAM policy and in audit logs, so it doubles as persistence only until someone reviews bindings.

## Tools

- **gcloud** (`add-iam-policy-binding`, `set-iam-policy`): the grant.
- **GCP-IAM-Privilege-Escalation** (Rhino): detects which `setIamPolicy` permissions the current member holds.

## References

- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: granting, changing, and revoking access](https://cloud.google.com/iam/docs/granting-changing-revoking-access)
