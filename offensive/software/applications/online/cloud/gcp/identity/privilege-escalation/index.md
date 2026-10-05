---
title: "Privilege escalation"
description: "The GCP IAM privilege-escalation catalog: resourcemanager setIamPolicy grants, custom-role edits, org-policy loosening, instance-metadata injection, and the actAs deploy-as-service-account paths."
keywords:
  - privilege escalation
  - setIamPolicy
  - actAs
  - custom role
  - org policy
  - service account
---

# Privilege escalation

GCP privilege escalation is a permission problem: a single dangerous permission turns a limited member into project or organization owner. The catalog below is grounded in the Rhino Security Labs method list and the GCP-IAM-Privilege-Escalation project, grouped by the primitive each path abuses.

## The paths

- **[setIamPolicy](set-iam-policy.md)**: grant yourself any role at a resource, project, folder, or organization you can administer.
- **[Custom role update](custom-role-update.md)**: add permissions to a custom role you already hold with `iam.roles.update`.
- **[Org policy](org-policy.md)**: loosen organization-policy constraints with `orgpolicy.policy.set` to unlock other escalations.
- **[Metadata startup script](metadata-startup-script.md)**: run code on an existing instance by writing `compute.instances.setMetadata` or an SSH key.
- **[actAs deployment](act-as-deployment/index.md)**: `iam.serviceAccounts.actAs` plus a resource-create verb to deploy code that runs as a privileged service account.

## Choosing a path

The most direct path is `setIamPolicy` if you hold it: it grants Owner in one call. Where you only hold `actAs` plus a deploy permission, pick the cheapest resource to create (a function or a Cloud Build is faster and quieter than a VM). Service-account impersonation primitives (`getAccessToken`, key creation) live under [service account impersonation](../service-account-impersonation/index.md) and often chain after one of these paths.

## References

- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud: GCP privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
