---
title: "Secret stores"
description: "Reading secrets from AWS secret stores: Secrets Manager, SSM Parameter Store SecureString values, and KMS-encrypted material."
keywords:
  - Secrets Manager
  - Parameter Store
  - KMS
  - SecureString
  - secrets
---

# Secret stores

AWS secret stores exist to hand credentials to applications, which makes them a direct source for an attacker who holds the read permission. A principal that can call `secretsmanager:GetSecretValue` or `ssm:GetParameter` with decryption is reading database passwords, API keys, and other credentials straight out of the account.

## What folds in here

- **[Secrets Manager](secrets-manager.md)**: `GetSecretValue` across stored secrets.
- **[Parameter Store](parameter-store.md)**: `GetParameter`/`GetParameters` over SecureString and plaintext values.
- **[KMS](kms.md)**: `Decrypt` and permissive key policies over envelope-encrypted data.

## Sweeping the stores

```bash
aws secretsmanager list-secrets --query 'SecretList[].Name'
aws ssm describe-parameters --query 'Parameters[].Name'
```

## References

- [HackTricks Cloud: AWS secrets](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: Secrets Manager GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html)
