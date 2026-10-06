---
title: "Federation"
order: 4
description: "Entering AWS through federated identity: SAML and OIDC provider trust, GitHub Actions OIDC, trusted third-party IdPs, and IAM Roles Anywhere certificates."
keywords:
  - federation
  - SAML
  - OIDC
  - GitHub Actions
  - Roles Anywhere
---

# Federation

Federation lets an external identity provider mint AWS role credentials without an IAM user. The trust is declared by an IAM **identity provider** object plus a role whose trust policy accepts tokens from it, scoped by `Condition` on claims such as `sub` and `aud`. When those conditions are loose, an attacker who controls or can forge a matching token assumes the role.

## What folds in here

- **[SAML](saml.md)**: `sts:AssumeRoleWithSAML` and over-broad SAML provider trust.
- **[OIDC](oidc.md)**: `sts:AssumeRoleWithWebIdentity` and weak `sub`/`aud` conditions on an OIDC provider.
- **[GitHub Actions](github-actions.md)**: the GitHub OIDC provider with a `sub` condition loose enough to accept other repositories or branches.
- **[Trusted IdP](trusted-idp.md)**: third-party IdPs (Auth0, Okta, Google) trusted as web-identity providers.
- **[Roles Anywhere](roles-anywhere.md)**: X.509 certificate trust that mints role credentials outside AWS.

## References

- [HackTricks Cloud: AWS federation abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [AWS: web identity federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_oidc.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
