---
title: "Cognito"
description: "Abusing Amazon Cognito: unauthenticated identity-pool role assumption, user-pool self-signup and attribute abuse, and forged or elevated token claims."
keywords:
  - Cognito
  - identity pool
  - user pool
  - token claims
  - unauthenticated
  - self-signup
---

# Cognito

Amazon Cognito is the consumer identity service behind many web and mobile apps, and it bridges application users to real IAM credentials. That bridge is the attack surface: a misconfigured identity pool hands AWS credentials to anonymous callers, and an open user pool lets anyone enroll.

## Pages

- **[Identity pool](identity-pool.md)**: exchanging an unauthenticated or guest identity for the pool's IAM role credentials.
- **[User pool](user-pool.md)**: open self-signup and writable attributes to gain or elevate access.
- **[Token claims](token-claims.md)**: abusing ID and access token claims to assume a more privileged identity.

## References

- [HackTricks Cloud: AWS Cognito](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cognito-enum/index.html)
- [Rhino Security Labs: Cognito research](https://rhinosecuritylabs.com/aws/)
