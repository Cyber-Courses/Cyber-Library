---
title: "Org policy: loosen constraints to unlock other escalations"
order: 2
description: "Disabling organization policy constraints with orgpolicy.policy.set to unlock service-account key creation, external sharing, and other blocked escalations."
keywords:
  - org policy
  - orgpolicy.policy.set
  - constraint
  - organization
  - policy bypass
---

# Org policy

Organization policy constraints block several escalation primitives by default: service-account key creation, cross-project service-account use, external IAM members, and VM external IPs. With `orgpolicy.policy.set` at a project, folder, or organization, you disable the constraint that is in your way and then run the escalation it was preventing.

## Loosen a constraint

```bash
# allow service-account key creation (then use Key creation for durable access)
gcloud resource-manager org-policies disable-enforce \
  constraints/iam.disableServiceAccountKeyCreation --project=<project>

# allow adding members outside the org domain
gcloud resource-manager org-policies delete \
  constraints/iam.allowedPolicyMemberDomains --project=<project>
```

Once `iam.disableServiceAccountKeyCreation` is off, follow [key creation](../service-account-impersonation/key-creation.md); once the domain restriction is gone, a [setIamPolicy](set-iam-policy.md) binding to an external attacker principal is accepted.

## Exploitation notes

- Org policy is a gate, not a grant: loosening it unlocks another path but does not itself give access, so it is always step one of a chain.
- Setting policy at the project scope overrides inheritance for that project only, which is quieter than changing the org default.
- The change is logged and visible in the policy, so pair it with the follow-on escalation quickly.

## Tools

- **gcloud** (`resource-manager org-policies`): disable or delete constraints.

## References

- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: organization policy constraints](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints)
