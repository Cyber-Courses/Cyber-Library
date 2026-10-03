---
title: "Policy manipulation"
description: "Granting yourself permissions by editing IAM policy: new default policy versions and attaching or inlining user, group, and role policies."
keywords:
  - policy manipulation
  - CreatePolicyVersion
  - AttachUserPolicy
  - PutUserPolicy
  - inline policy
  - managed policy
---

# Policy manipulation

When a principal can write IAM policy, it can write itself to administrator. These are the most direct escalations in AWS: a single `iam:` write, no new resource, no host. They split into three shapes: changing which **version** of a managed policy is active, **attaching** a managed policy, and **inlining** a policy document.

## Managed policy versions

- **[CreatePolicyVersion](create-policy-version.md)**: publish a new default version of a policy you can edit, granting full access.
- **[SetDefaultPolicyVersion](set-default-policy-version.md)**: activate an existing, more permissive version.

## Attaching managed policies

- **[AttachUserPolicy](attach-user-policy.md)**: attach `AdministratorAccess` to a user you control.
- **[AttachGroupPolicy](attach-group-policy.md)**: attach a privileged policy to a group you belong to.
- **[AttachRolePolicy](attach-role-policy.md)**: attach a privileged policy to a role you can assume.

## Inlining policy documents

- **[PutUserPolicy](put-user-policy.md)**: inline an allow-all document onto a user.
- **[PutGroupPolicy](put-group-policy.md)**: inline an allow-all document onto a group.
- **[PutRolePolicy](put-role-policy.md)**: inline an allow-all document onto a role.

## References

- [Rhino Security Labs: AWS privilege escalation (policy paths)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
