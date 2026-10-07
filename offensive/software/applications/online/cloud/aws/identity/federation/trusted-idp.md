---
title: "Trusted IdP: riding third-party identity trust into AWS"
order: 4
description: "Abusing trust placed in a third-party identity provider or SaaS vendor to obtain federated AWS access."
keywords:
  - identity provider
  - federation
  - trust
  - third party
  - SSO
---

# Trusted IdP

An account often federates to a third-party identity provider (Okta, Auth0, Google, a corporate IdP) as a SAML or OIDC provider. The AWS trust is only as strong as that external IdP: compromise an account at the IdP that maps to an AWS role, or the IdP itself, and you inherit the mapped AWS access.

## Mapping the trust

```bash
aws iam list-saml-providers
aws iam list-open-id-connect-providers
# for each role, read the trust to see which provider and claims it accepts
aws iam get-role --role-name <role> --query 'Role.AssumeRolePolicyDocument'
```

## Exploitation notes

- The weak link is frequently the IdP's own authentication: an IdP account with a role mapping and weak MFA is a path into AWS.
- Role mappings driven by an IdP group or attribute mean adding yourself to that group at the IdP grants the AWS role.
- Concrete mechanics follow [SAML](saml.md) or [OIDC](oidc.md) depending on the provider type.

## Tools

- **AWS CLI** (`iam list-saml-providers` / `list-open-id-connect-providers`).
- IdP-specific tooling for the provider in use.

## References

- [AWS: identity providers and federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html)
- [HackTricks Cloud: AWS federation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
