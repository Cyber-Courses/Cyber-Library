---
title: "OIDC: AssumeRoleWithWebIdentity and weak claim conditions"
description: "Abusing an IAM OIDC identity provider through sts:AssumeRoleWithWebIdentity when the role trust policy's sub and aud conditions are missing or too broad."
keywords:
  - OIDC
  - AssumeRoleWithWebIdentity
  - web identity
  - sub claim
  - aud claim
---

# OIDC

An IAM OIDC provider trusts tokens signed by an external issuer. A role tied to it is assumed with `sts:AssumeRoleWithWebIdentity`, and the only thing standing between an attacker and the role is the trust policy's `Condition` block on the token's `sub` (subject) and `aud` (audience) claims. If those are absent or wildcarded, any token the issuer will mint, including for an identity the attacker controls, assumes the role.

## Assuming with a web-identity token

```bash
aws sts assume-role-with-web-identity \
  --role-arn arn:aws:iam::<acct>:role/<role> \
  --role-session-name s \
  --web-identity-token "$OIDC_JWT" --query Credentials
```

## Reading the trust condition

```bash
aws iam get-role --role-name <r> --query 'Role.AssumeRolePolicyDocument'
# look for the OIDC provider as Principal.Federated and the StringEquals/StringLike
# conditions on "<issuer>:sub" and "<issuer>:aud"
```

A trust with `StringLike` and a `*` in `sub`, or no `sub` condition at all, is assumable by any subject the issuer serves.

## Exploitation notes

- The audience check is only as strong as the `aud` condition; a missing `aud` lets a token minted for a different application through.
- For managed issuers you do not control, you still win when the `sub` condition is loose enough to match an identity you can obtain from that issuer.
- GitHub Actions is the most common concrete case and has its own [page](github-actions.md).

## Tools

- **AWS CLI** (`sts assume-role-with-web-identity`): the assumption.
- **jwt_tool**: inspect and craft the claims in a web-identity JWT.

## References

- [HackTricks Cloud: OIDC web-identity abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [AWS: AssumeRoleWithWebIdentity](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
