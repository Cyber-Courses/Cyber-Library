---
title: "AttachUserPolicy: attach AdministratorAccess to your user"
description: "iam:AttachUserPolicy to attach AdministratorAccess or another privileged managed policy to a user you control."
keywords:
  - AttachUserPolicy
  - managed policy
  - AdministratorAccess
  - user
  - attach
---

# AttachUserPolicy

`iam:AttachUserPolicy` attaches a managed policy to a user. If you hold it for your own user, attach the AWS-managed `AdministratorAccess` and you are administrator in one call.

## Attach admin

```bash
aws iam attach-user-policy --user-name <you> \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

## Exploitation notes

- `AdministratorAccess` is an AWS-managed policy present in every account, so no policy creation is needed.
- Works only against users; the role and group equivalents are [AttachRolePolicy](attach-role-policy.md) and [AttachGroupPolicy](attach-group-policy.md).

## Tools

- **AWS CLI** (`iam attach-user-policy`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (AttachUserPolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
