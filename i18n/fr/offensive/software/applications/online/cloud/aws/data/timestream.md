---
title: "Timestream : interroger des bases de données de séries temporelles"
order: 12
description: "Interroger des bases de données de séries temporelles Timestream auxquelles l'appelant ne devrait pas avoir accès."
keywords:
  - Timestream
  - série temporelle
  - requête
  - base de données
  - accès aux données
---

# Timestream

Timestream est la base de données de séries temporelles d'AWS, accédée entièrement via IAM sans surface réseau. Un principal détenant `timestream:Select` interroge toute base de données et table que la stratégie autorise, ce qui est fréquemment plus large que prévu car les permissions Timestream sont souvent accordées au niveau du service. Les données, télémétrie opérationnelle et IoT, peuvent révéler l'infrastructure, les comportements et les volumes utiles à l'engagement au sens large.

## Interroger

```bash
aws timestream-write list-databases
aws timestream-write list-tables --database-name <db>
aws timestream-query query \
  --query-string 'SELECT * FROM "db"."table" LIMIT 100'
```

## Notes d'exploitation

- Les points de terminaison de requête se découvrent avec `timestream-query describe-endpoints` ; le SDK s'en charge, mais notez-le lors d'un appel direct à l'API.
- Les autorisations `timestream:*` au niveau du service sont courantes et donnent un accès en lecture à toutes les bases de données du compte.
- Les données sont de la télémétrie en ajout seul, de sorte que la valeur réside dans le renseignement plutôt que dans l'altération.

## Outils

- **AWS CLI** (`timestream-query query`, `timestream-write list-databases`).

## Références

- [AWS: Timestream access control](https://docs.aws.amazon.com/timestream/latest/developerguide/security-iam.html)
- [HackTricks Cloud: AWS Timestream](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable cloud data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
