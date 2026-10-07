---
title: "Identity Store: adding users and group membership"
order: 2
description: "Abusing the Identity Center Identity Store to add or modify users and group membership and inherit their access."
keywords:
  - Identity Store
  - IAM Identity Center
  - user
  - group membership
  - SSO
---

# Identity Store

The Identity Store holds Identity Center's users and groups. Because permission-set assignments target users and groups, write access to the store lets you add yourself to a group that already carries assignments, or create a user mapped to privileged access.

## Join a privileged group

```bash
aws identitystore list-groups --identity-store-id <store-id>
aws identitystore create-group-membership --identity-store-id <store-id> \
  --group-id <privileged-group-id> \
  --member-id UserId=<your-user-id>
```

## Exploitation notes

- Target groups that already hold account assignments: membership inherits those permission sets without touching the assignments themselves.
- When Identity Center syncs from an external IdP (SCIM), the external directory is the real control point; changes may be overwritten on the next sync, so this is strongest on an internally-managed store.
- Pair with [permission set](permission-set.md): the store decides who, the permission set decides what.

## Tools

- **AWS CLI** (`identitystore` commands).

## References

- [HackTricks Cloud: AWS Identity Center](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-iam-and-sts-enum.html)
- [AWS: Identity Store](https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/welcome.html)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
