---
title: "GCP identity"
description: "Attacking Google Cloud IAM: enumerating members and roles, the privilege-escalation catalog (setIamPolicy, actAs deploys, metadata), service-account impersonation, and workload identity federation."
keywords:
  - Cloud IAM
  - service account
  - privilege escalation
  - impersonation
  - actAs
---

# Identity

Cloud IAM is where GCP attacks are won. A binding grants a member (user, group, or **service account**) a role at a scope, and the whole game is turning a permission you hold into control of a more privileged service account. Two mechanisms dominate: **impersonation**, where a permission like `iam.serviceAccounts.getAccessToken` mints another account's token directly, and **actAs**, where `iam.serviceAccounts.actAs` plus a resource-create permission lets you deploy code (a VM, function, build, or job) that runs as a privileged account.

Privilege escalation lives here because in GCP it is a permission problem: a single dangerous permission (`resourcemanager.projects.setIamPolicy`, `iam.serviceAccountKeys.create`, `iam.serviceAccounts.actAs` with a deploy verb) promotes a limited member to project or organization owner.

## What folds in here

- **[Enumeration](enumeration.md)**: resolving members, roles, and which service accounts you can reach, with `gcloud` and IAM-graph tooling.
- **[Privilege escalation](privilege-escalation/index.md)**: `setIamPolicy`, custom-role updates, org policy, metadata startup scripts, and the `actAs` deploy-as-service-account catalog.
- **[Service account impersonation](service-account-impersonation/index.md)**: `getAccessToken`, `signJwt` and `signBlob`, Token Creator grants, implicit delegation, and key creation.
- **[Federation](federation.md)**: workload identity federation from an external OIDC or AWS identity into a GCP service account.

## References

- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud: GCP privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
