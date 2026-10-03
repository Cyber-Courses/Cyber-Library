---
title: "AWS identity"
description: "The core AWS attack surface: the IAM graph of users, roles, and policies, role assumption and iam:PassRole, policy abuse, and the privilege-escalation paths that turn a limited principal into an administrative one."
keywords:
  - IAM
  - AssumeRole
  - PassRole
  - privilege escalation
  - trust policy
---

# Identity

IAM is where AWS attacks are won. Every call is authorized against the principal's policies, so the questions are: what can this principal do, which roles can it **assume**, and which permissions let it **escalate**. Privilege escalation lives here because in AWS it is a permission problem, not a host exploit: a single dangerous action (or a weak role **trust policy**) promotes a limited identity to administrator.

The pages here cover the permission-escalation primitives (`iam:PassRole` with a compute-creation action, `iam:CreatePolicyVersion`, `iam:AttachUserPolicy`, `sts:AssumeRole` against a permissive trust policy, and the rest of the known paths), role assumption across accounts, and the policy and trust-relationship analysis that PMapper and Pacu automate.

## What folds in here

- **Privilege escalation**: the IAM permission paths to administrator.
- **Federation**: SAML/OIDC role assumption and GitHub-Actions-style workload identity.
- **Lateral movement** across accounts via `AssumeRole` is cross-referenced from [networking](../networking/index.md) and compute, but the assume-role mechanics live here.

## References

- [Rhino Security Labs: AWS IAM privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [PMapper (NCC Group)](https://github.com/nccgroup/PMapper)
- [HackTricks Cloud: AWS IAM privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/index.html)
