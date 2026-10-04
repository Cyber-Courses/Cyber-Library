---
title: "SetDefaultPolicyVersion: activate a more permissive existing version"
description: "iam:SetDefaultPolicyVersion to activate an existing but more permissive version of a managed policy."
keywords:
  - SetDefaultPolicyVersion
  - managed policy
  - policy version
  - rollback
  - admin
---

# SetDefaultPolicyVersion

A managed policy may already carry an older, broader version that is no longer the default. `iam:SetDefaultPolicyVersion` activates any stored version, so if a policy attached to you has a permissive non-default version in its history, you switch to it without writing a new document.

## List versions and switch

```bash
aws iam list-policy-versions --policy-arn arn:aws:iam::<acct>:policy/<policy>
# find a version whose document is broader than the current default
aws iam set-default-policy-version \
  --policy-arn arn:aws:iam::<acct>:policy/<policy> --version-id v2
```

## Exploitation notes

- This needs only `iam:SetDefaultPolicyVersion`, a narrower permission than `CreatePolicyVersion`, and leaves no new policy document behind.
- Inspect each stored version's document with `get-policy-version` to find the most permissive one before switching.

## Tools

- **AWS CLI** (`iam list-policy-versions` / `set-default-policy-version`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (SetDefaultPolicyVersion)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
