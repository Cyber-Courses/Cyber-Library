---
title: "GitHub Actions: assuming roles through a loose OIDC subject"
description: "Abusing a loosely-scoped GitHub Actions OIDC trust (wildcard subject claim) to assume a role from an attacker-controlled workflow."
keywords:
  - GitHub Actions
  - OIDC
  - sub claim
  - workflow
  - AssumeRoleWithWebIdentity
---

# GitHub Actions

GitHub Actions can authenticate to AWS through the GitHub OIDC provider, assuming a role with `sts:AssumeRoleWithWebIdentity` and no stored secret. The role's trust policy is meant to pin the token's `sub` claim to a specific `repo:org/name:...`. When that condition is wildcarded or scoped only to the org, a workflow in any matching repository, including a fork or a repo you create, assumes the role.

## The loose trust

```bash
aws iam get-role --role-name <role> --query 'Role.AssumeRolePolicyDocument'
# look for token.actions.githubusercontent.com:sub with StringLike "repo:org/*"
# or a condition on :aud only, with no :sub pin
```

A `sub` like `repo:org/*:ref:refs/heads/*` accepts any branch of any repo in the org; a missing `sub` accepts any repo GitHub will mint a token for.

## Assuming from a workflow

A workflow you control in a matching repo requests the OIDC token and assumes:

```yaml
permissions: { id-token: write }
steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::<acct>:role/<role>
      aws-region: us-east-1
```

## Exploitation notes

- Org-only scoping is the common mistake: a public org lets outsiders open a repo that satisfies `repo:org/*`.
- The `aud` claim alone is not a scope; without a `sub` condition any token the provider issues is accepted.
- See [OIDC](oidc.md) for the general web-identity mechanics behind this.

## Tools

- **aws-actions/configure-aws-credentials**: the standard assumption action.
- **AWS CLI** (`sts assume-role-with-web-identity`) with a token obtained from the Actions runtime.

## References

- [GitHub: configuring OpenID Connect in AWS](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- [Rhino Security Labs: AWS privilege escalation](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
