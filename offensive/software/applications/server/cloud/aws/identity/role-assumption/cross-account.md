---
title: "Cross-account: assuming roles that trust another account"
description: "Abusing IAM role trust policies that name a whole external account or root, letting any principal you hold there assume into the target account."
keywords:
  - cross-account
  - AssumeRole
  - trust policy
  - root principal
  - lateral movement
---

# Cross-account

A role's trust policy can name a **principal in another account**. When it names the account root (`arn:aws:iam::<acct>:root`) rather than a specific role, *any* principal in that account with `sts:AssumeRole` can assume the role, which turns a foothold in one account into access in another.

## Spotting the trust

```json
{ "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::111111111111:root" },
  "Action": "sts:AssumeRole" }
```

`root` here means the whole account, not only its root user: it delegates the authorization decision to account `111111111111`'s own IAM.

## Assuming across

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::<target>:role/<role> \
  --role-session-name x --query Credentials
```

## Exploitation notes

- Enumerate trust documents account-wide with `get-account-authorization-details`, then filter for `Principal.AWS` values that are account roots or wildcards.
- Role chaining works: assume role A in account 1, then use it to assume role B in account 2, following the trust graph PMapper draws.
- A session assumed by chaining is capped at one hour and cannot be renewed by re-chaining, which matters for long operations.

## Tools

- **AWS CLI** (`sts assume-role`): the assumption.
- **PMapper**: cross-account edges when graphs for both accounts are loaded.
- **Pacu** (`iam__enum_assume_role`): brute candidate role ARNs for assumable roles.

## References

- [HackTricks Cloud: cross-account trust abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [PMapper (NCC Group)](https://github.com/nccgroup/PMapper)
