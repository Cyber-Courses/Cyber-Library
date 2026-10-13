---
title: "Confused deputy: assuming vendor roles without the external ID"
order: 2
description: "Abusing a role whose trust policy trusts a third party without an external ID, riding the deputy's access into the account."
keywords:
  - confused deputy
  - external ID
  - trust policy
  - third party
  - AssumeRole
---

# Confused deputy

Third-party SaaS vendors are given a role whose trust policy names the vendor's AWS account as principal. AWS designed the **external ID** condition to scope that trust to your specific tenant; when it is missing, any customer of that vendor (or anyone who can make the vendor assume the role) rides the vendor's access into the account. This is the confused-deputy problem.

## Spotting the gap

```bash
aws iam get-role --role-name <vendor-role> --query 'Role.AssumeRolePolicyDocument'
# Principal.AWS = the vendor account, and NO Condition on sts:ExternalId
```

A trust that names a vendor account with no `StringEquals` on `sts:ExternalId` is assumable by that vendor on behalf of any tenant, including one an attacker controls in the vendor's product.

## Exploitation notes

- The attack runs through the vendor's platform: you configure the vendor to point at the victim role, and the vendor (the deputy) assumes it for you.
- Even with an external ID, a guessable or leaked value reopens the path, so weak external IDs are as good as none.
- Cross-reference [cross-account](cross-account.md) for the simpler case where the trust names a whole account directly.

## Tools

- **AWS CLI** (`iam get-role`): read the trust document.
- **PMapper**: flags roles trusting external accounts.

## References

- [AWS: the confused deputy problem](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html)
- [HackTricks Cloud: cross-account and confused deputy](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
