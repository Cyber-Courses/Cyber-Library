---
title: "Lake Formation: credential vending for governed tables"
description: "Abusing Lake Formation permissions and credential vending to read governed data-lake tables."
keywords:
  - Lake Formation
  - data lake
  - permissions
  - credential vending
  - table
---

# Lake Formation

Lake Formation governs access to data-lake tables and vends short-lived S3 credentials scoped to what a principal is granted. The attack is to hold or grant yourself a Lake Formation permission and then call the vending API, which returns temporary credentials for the underlying S3 data, bypassing the bucket's own policy. A principal that can administer Lake Formation grants can widen its own access to any governed table.

## Vending credentials for a table

```bash
aws lakeformation list-permissions
aws lakeformation get-temporary-glue-table-credentials \
  --table-arn <table-arn> --supported-permission-types COLUMN_PERMISSION
# the response contains temporary S3 credentials for the governed data
```

## Granting yourself access

```bash
aws lakeformation grant-permissions \
  --principal DataLakePrincipalIdentifier=<your-arn> \
  --resource '{"Table":{"DatabaseName":"db","Name":"t"}}' \
  --permissions SELECT
```

## Exploitation notes

- Credential vending returns real S3 credentials for the governed location, so it reads data even when the bucket policy would deny the caller directly.
- `lakeformation:GrantPermissions` with admin scope is a data-plane privilege escalation: grant then vend.
- Governed tables often front the most sensitive lake data, which is why the vending path is worth the extra step.

## Tools

- **AWS CLI** (`lakeformation get-temporary-glue-table-credentials`, `grant-permissions`).

## References

- [AWS: Lake Formation credential vending](https://docs.aws.amazon.com/lake-formation/latest/dg/credential-vending.html)
- [HackTricks Cloud: AWS Lake Formation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
