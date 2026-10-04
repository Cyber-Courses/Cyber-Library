---
title: "Identity Center"
description: "Abusing AWS IAM Identity Center (SSO): permission-set assignment and the Identity Store to reach accounts across the organization."
keywords:
  - IAM Identity Center
  - AWS SSO
  - permission set
  - Identity Store
  - organization
  - assignment
---

# Identity Center

AWS IAM Identity Center (formerly AWS SSO) is the front door to an entire organization: it maps workforce users to **permission sets** in member accounts. Control over its assignments or its user directory turns one foothold into access across every account it manages.

## Pages

- **[Permission set](permission-set.md)**: assigning or editing a permission set to grant yourself a role in a target account.
- **[Identity Store](identity-store.md)**: adding or modifying users and group membership to inherit their access.

## References

- [HackTricks Cloud: AWS IAM Identity Center](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-iam-and-sts-enum.html)
- [AWS: IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
