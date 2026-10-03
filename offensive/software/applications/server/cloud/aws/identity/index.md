---
title: "AWS identity"
description: "AWS identity and access attacks: enumerating IAM principals and policies, the full privilege-escalation path catalog, cross-account role assumption and federation, and the Cognito, Identity Center, and Organizations identity layers."
keywords:
  - IAM
  - AssumeRole
  - PassRole
  - privilege escalation
  - federation
---

# Identity

IAM is where AWS attacks are won. Every API call is authorized against the calling principal's policies, so the questions are always the same: what can this principal do, which roles can it **assume**, and which permissions let it **escalate**. Privilege escalation lives here because in AWS it is a permission problem, not a host exploit: a single dangerous action, or a weak role **trust policy**, promotes a limited identity to administrator.

Beyond classic IAM users and roles, AWS exposes several distinct identity layers that each carry their own paths in: the **Cognito** consumer identity service, **IAM Identity Center** (SSO) across an organization, and **Organizations** itself, where one assumed role reaches every member account.

## What folds in here

- **[Enumeration](enumeration.md)**: mapping principals, policies, and trust relationships, and resolving the current principal's effective permissions.
- **[Privilege escalation](privilege-escalation/index.md)**: the full catalog of IAM paths to administrator, from `iam:PassRole` to policy and trust-policy rewrites.
- **[Role assumption](role-assumption/index.md)**: `sts:AssumeRole` across accounts and confused-deputy abuse of permissive trust policies.
- **[Federation](federation/index.md)**: SAML and OIDC provider trust, GitHub Actions OIDC, trusted third-party IdPs, and IAM Roles Anywhere.
- **[Cognito](cognito/index.md)**: unauthenticated identity-pool roles, user-pool self-signup, and token-claim abuse.
- **[Identity Center](identity-center/index.md)**: permission-set assignment and the Identity Store to reach accounts across the organization.
- **[Organizations](organizations/index.md)**: the management-created member-account role and service control policies.

Enumeration is folded into each surface across AWS, but the IAM graph that drives everything is mapped here with **Pacu** and **PMapper**.

## References

- [Rhino Security Labs: AWS IAM privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [PMapper (NCC Group)](https://github.com/nccgroup/PMapper)
- [HackTricks Cloud: AWS IAM privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/index.html)
