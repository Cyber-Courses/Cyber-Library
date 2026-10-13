---
title: "Redshift : atteindre l'entrepôt via des identifiants temporaires et l'exposition"
order: 4
description: "Accéder à des clusters Redshift via des points de terminaison exposés, des identifiants temporaires et des autorisations de base de données trop larges."
keywords:
  - Redshift
  - cluster
  - GetClusterCredentials
  - base de données
  - entrepôt de données
---

# Redshift

Redshift est l'entrepôt de données, il concentre donc exactement les données qui valent la peine d'être prises. Un principal disposant de `redshift:GetClusterCredentials` forge un utilisateur et un mot de passe de base de données temporaires pour un cluster, créant automatiquement l'utilisateur lorsque c'est autorisé, ce qui transforme un point d'ancrage IAM en accès à l'entrepôt sans aucun mot de passe stocké. Les clusters accessibles publiquement avec des groupes de sécurité laxistes sont joignables directement, et la Redshift Data API exécute du SQL via le seul plan de contrôle.

## Forger des identifiants et se connecter

```bash
aws redshift describe-clusters \
  --query 'Clusters[?PubliclyAccessible==`true`].[ClusterIdentifier,Endpoint.Address]'

aws redshift get-cluster-credentials \
  --cluster-identifier <c> --db-user loot --auto-create --db-name dev
psql "host=<ep> port=5439 dbname=dev user=IAM:loot password=<temp>"
```

## Interroger via la Data API

```bash
aws redshift-data execute-statement --cluster-identifier <c> \
  --database dev --sql 'select * from users limit 100'
aws redshift-data get-statement-result --id <stmt-id>
```

## Notes d'exploitation

- `GetClusterCredentials` avec `--auto-create` et une autorisation `DbGroups` large peut vous placer dans un groupe de base de données privilégié.
- La Data API ne nécessite aucun chemin réseau vers le cluster, seulement IAM, elle fonctionne donc partout où les identifiants fonctionnent.
- Redshift Spectrum lit S3 via des schémas externes, étendant l'accès aux données du lac référencées par l'entrepôt.

## Outils

- **AWS CLI** (`redshift get-cluster-credentials`, `redshift-data execute-statement`).
- **psql** : client natif une fois les identifiants forgés.

## Références

- [AWS: GetClusterCredentials](https://docs.aws.amazon.com/redshift/latest/APIReference/API_GetClusterCredentials.html)
- [HackTricks Cloud: AWS Redshift](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable warehouses and data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
