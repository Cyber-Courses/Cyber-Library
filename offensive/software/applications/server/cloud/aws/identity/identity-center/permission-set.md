---
title: "Permission set: granting yourself a role in a target account"
description: "Assigning or editing an Identity Center permission set to grant yourself a role in a target account."
keywords:
  - permission set
  - IAM Identity Center
  - assignment
  - account
  - SSO
---

# Permission set

A permission set is a template that Identity Center provisions as an IAM role in a member account. With the SSO admin permissions, you assign an existing high-privilege permission set (or edit one) to your own user or group in a target account, then log in through the SSO portal as that role.

## Assign a permission set

```bash
aws sso-admin list-permission-sets --instance-arn <instance-arn>
aws sso-admin create-account-assignment --instance-arn <instance-arn> \
  --permission-set-arn <ps-arn> \
  --principal-type USER --principal-id <your-identitystore-user-id> \
  --target-type AWS_ACCOUNT --target-id <account-id>
```

Then retrieve credentials through the SSO portal or `aws sso login`.

## Exploitation notes

- Editing a permission set's inline or managed policies pushes the change to every account it is assigned in, a broad escalation.
- The management or delegated-admin account holds these permissions; reaching it (for example through [Organizations](../organizations/member-account-role.md)) unlocks the whole estate.
- Assignments provision real IAM roles, so the granted access behaves like any assumed role afterward.

## Tools

- **AWS CLI** (`sso-admin` commands, `aws sso login`).
- **Pacu**: Identity Center enumeration.

## References

- [HackTricks Cloud: AWS SSO and Identity Center](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-iam-and-sts-enum.html)
- [AWS: permission sets](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html)
