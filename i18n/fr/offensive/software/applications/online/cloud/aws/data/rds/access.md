---
title: "Accès : atteindre des instances RDS en service via l'exposition et une authentification faible"
order: 2
description: "Atteindre des instances RDS via des points de terminaison exposés publiquement, des identifiants faibles ou l'authentification IAM RDS."
keywords:
  - RDS
  - point de terminaison
  - authentification IAM
  - identifiants de base de données
  - accès
---

# Accès

Une instance RDS en service est joignable dès lors que son point de terminaison est accessible publiquement et que son groupe de sécurité autorise votre source, ou lorsque vous pouvez forger des identifiants pour elle. Les identifiants maîtres et applicatifs se trouvent fréquemment dans [Secrets Manager](../../credentials/secret-stores/secrets-manager.md) ou dans un script user-data d'EC2, et lorsque l'authentification IAM RDS est activée, un principal disposant de `rds-db:connect` forge un jeton éphémère sans aucun mot de passe.

## Trouver des instances joignables

```bash
aws rds describe-db-instances \
  --query 'DBInstances[?PubliclyAccessible==`true`].[DBInstanceIdentifier,Endpoint.Address,Endpoint.Port]'
```

## Se connecter

```bash
# IAM authentication: no password, mint a token
TOKEN=$(aws rds generate-db-auth-token --hostname <ep> --port 3306 --username iam_user)
mysql -h <ep> -P 3306 -u iam_user --password="$TOKEN" --enable-cleartext-plugin

# or with credentials recovered from Secrets Manager / user-data
psql "host=<ep> port=5432 dbname=app user=admin password=<pw>"
```

## Notes d'exploitation

- `PubliclyAccessible` plus un groupe de sécurité ouvert sur `0.0.0.0/0` est l'exposition courante ; même sans cela, une instance est joignable depuis une machine EC2 compromise dans le même VPC.
- Les jetons d'authentification IAM RDS durent 15 minutes mais peuvent être reforgés indéfiniment tant que la permission tient.
- Les identifiants maîtres récupérés permettent aussi un changement de mot de passe et un accès durable ; mettez cela en balance avec le bruit lié à l'altération du compte.

## Outils

- **AWS CLI** (`rds describe-db-instances`, `generate-db-auth-token`).
- **mysql / psql / sqlcmd** : clients natifs une fois l'instance joignable.

## Références

- [AWS: IAM database authentication](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html)
- [HackTricks Cloud: AWS RDS access](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable RDS instances](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
