---
title: "Role assumption"
order: 3
description: "Assuming AWS IAM roles through sts:AssumeRole: cross-account access and confused-deputy abuse of over-broad trust policies and missing external IDs."
keywords:
  - AssumeRole
  - STS
  - cross-account
  - confused deputy
  - trust policy
---

# Role assumption

A role is assumable by whoever its **trust policy** allows. `sts:AssumeRole` returns temporary credentials for the role, and because trust policies are frequently written too broadly (a whole account as principal, a wildcard, or a third-party account with no external ID), role assumption is both a lateral-movement and an escalation primitive.

## Assuming a role

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::<acct>:role/<role> \
  --role-session-name s --query Credentials
# export AccessKeyId / SecretAccessKey / SessionToken and continue as the role
```

## What folds in here

- **[Cross-account](cross-account.md)**: trust policies that name another account (or `root`), letting any principal there assume in.
- **[Confused deputy](confused-deputy.md)**: third-party-vendor roles assumable without the external ID that was meant to scope them.

## Finding assumable roles

Read trust documents from the [enumeration](../enumeration.md) dump and look for `Principal` values broader than a single role ARN, and for `sts:AssumeRole` statements missing a `Condition` on `sts:ExternalId`.

## References

- [HackTricks Cloud: AssumeRole and trust policies](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [AWS: the confused deputy problem](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
