---
title: "Identity pool: credentials for an unauthenticated identity"
description: "Exchanging an unauthenticated or guest Cognito identity for the identity pool's IAM role credentials."
keywords:
  - Cognito
  - identity pool
  - unauthenticated
  - guest
  - IAM role
---

# Identity pool

A Cognito identity pool maps app users (and, when enabled, **unauthenticated guests**) to IAM roles and returns real AWS credentials. If the pool allows unauthenticated identities, anyone with the pool ID gets the guest role's credentials, and those roles are frequently over-permissioned.

## Get guest credentials

```bash
# the pool id is often visible in the app's JS or mobile bundle
aws cognito-identity get-id --identity-pool-id <region>:<pool-id>
aws cognito-identity get-credentials-for-identity --identity-id <id>
# export the returned keys and enumerate what the guest role can do
```

## Exploitation notes

- Pool IDs are client-side values, so treat any front end using Cognito as a source for them.
- The guest role's reach is the prize: enumerate it with [identity enumeration](../enumeration.md), since these roles often grant S3 or DynamoDB access far beyond what the app needs.
- Authenticated-identity credentials follow the same calls once you hold a user-pool token, which links to [token claims](token-claims.md).

## Tools

- **AWS CLI** (`cognito-identity get-id` / `get-credentials-for-identity`).
- **Pacu**: Cognito enumeration modules.

## References

- [HackTricks Cloud: AWS Cognito identity pools](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cognito-enum/index.html)
- [AWS: Cognito identity pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-identity.html)
- [cognito-scanner](https://pypi.org/project/cognito-scanner/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
