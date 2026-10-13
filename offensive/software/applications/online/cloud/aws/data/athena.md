---
title: "Athena: querying S3 data through the catalog"
order: 8
description: "Querying data in S3 through Athena, reaching tables and locations the caller should not see."
keywords:
  - Athena
  - query
  - S3
  - Data Catalog
  - data access
---

# Athena

Athena runs SQL over data in S3 through the Glue Data Catalog. A principal with Athena and the underlying S3 read permissions queries any table the catalog knows about, which is often broader than the buckets they were meant to see. Where a table is not catalogued, `CREATE EXTERNAL TABLE` points Athena at an arbitrary S3 location and reads it directly.

## Querying

```bash
aws athena start-query-execution \
  --query-string 'SELECT * FROM logs.access LIMIT 100' \
  --result-configuration OutputLocation=s3://<reachable-bucket>/out/
aws athena get-query-results --query-execution-id <id>
```

## Reaching uncatalogued data

```sql
CREATE EXTERNAL TABLE loot (line string)
LOCATION 's3://<target-bucket>/path/';
SELECT * FROM loot;
```

## Exploitation notes

- Athena's reach is the union of the catalog and the caller's S3 permissions; a broad `s3:GetObject` plus Athena reads data across many buckets through one interface.
- Query results land in the configured output bucket, so a readable output location is itself an exfiltration channel.
- Workgroup settings can force an output location, which is worth reading before assuming where results go.

## Tools

- **AWS CLI** (`athena start-query-execution`, `get-query-results`).
- **awswrangler**: scripted Athena queries and result retrieval.

## References

- [AWS: Athena and S3 permissions](https://docs.aws.amazon.com/athena/latest/ug/security-iam-athena.html)
- [HackTricks Cloud: AWS Athena](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: finding exploitable paths across cloud data services](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
