---
title: "Magasins de secrets"
order: 4
description: "Lire des secrets dans les magasins de secrets AWS : Secrets Manager, valeurs SecureString de SSM Parameter Store et données chiffrées par KMS."
keywords:
  - Secrets Manager
  - Parameter Store
  - KMS
  - SecureString
  - secrets
---

# Magasins de secrets

Les magasins de secrets AWS existent pour fournir des identifiants aux applications, ce qui en fait une source directe pour un attaquant détenant le droit de lecture. Un principal capable d'appeler `secretsmanager:GetSecretValue` ou `ssm:GetParameter` avec déchiffrement lit les mots de passe de bases de données, les clés d'API et d'autres identifiants directement dans le compte.

## Ce qui est regroupé ici

- **[Secrets Manager](secrets-manager.md)** : `GetSecretValue` sur les secrets stockés.
- **[Parameter Store](parameter-store.md)** : `GetParameter`/`GetParameters` sur les valeurs SecureString et en clair.
- **[KMS](kms.md)** : `Decrypt` et stratégies de clés trop permissives sur des données chiffrées par enveloppe.

## Balayer les magasins

```bash
aws secretsmanager list-secrets --query 'SecretList[].Name'
aws ssm describe-parameters --query 'Parameters[].Name'
```

## Références

- [HackTricks Cloud: AWS secrets](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: Secrets Manager GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
