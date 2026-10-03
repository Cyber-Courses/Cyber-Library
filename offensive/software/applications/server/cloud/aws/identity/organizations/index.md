---
title: "Organizations"
description: "Pivoting across an AWS Organization: assuming the management-created OrganizationAccountAccessRole in member accounts and manipulating service control policies."
keywords:
  - AWS Organizations
  - OrganizationAccountAccessRole
  - SCP
  - management account
  - member account
  - cross-account
---

# Organizations

AWS Organizations binds many accounts under one management account. That structure is a pivot: the management account holds a default role into every member, and it sets the service control policies that bound them all. Reaching the management account, or an account that can assume into others, cascades across the estate.

## Pages

- **[Member account role](member-account-role.md)**: assuming the default OrganizationAccountAccessRole the management account creates in each member.
- **[SCP manipulation](scp-manipulation.md)**: editing service control policies to lift organization-wide guardrails.

## References

- [HackTricks Cloud: AWS Organizations](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-organizations-enum.html)
- [AWS: Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)
