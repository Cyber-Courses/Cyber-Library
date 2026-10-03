---
title: "Member account role: assuming OrganizationAccountAccessRole"
description: "sts:AssumeRole into the OrganizationAccountAccessRole that the management account creates by default in every member account."
keywords:
  - OrganizationAccountAccessRole
  - member account
  - AssumeRole
  - management account
  - Organizations
---

# Member account role

When an account is created in or invited to an Organization, AWS provisions a role named **OrganizationAccountAccessRole** in the member, trusting the management account with administrator permissions. From the management account (or any principal there allowed to assume it), you drop into any member account as administrator.

## Assume into a member

```bash
aws organizations list-accounts --query 'Accounts[].Id'
aws sts assume-role \
  --role-arn arn:aws:iam::<member-acct>:role/OrganizationAccountAccessRole \
  --role-session-name s
```

## Exploitation notes

- The role name is the default; some orgs rename it, so enumerate roles in a member if the default is absent.
- This makes the management account the highest-value target in an Organization: it is administrator everywhere by design.
- Accounts created by the org have the role automatically; invited accounts may not, so coverage varies.

## Tools

- **AWS CLI** (`organizations list-accounts`, `sts assume-role`).
- **PMapper** / **Pacu**: cross-account path discovery.

## References

- [AWS: OrganizationAccountAccessRole](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html)
- [HackTricks Cloud: AWS Organizations](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-organizations-enum.html)
