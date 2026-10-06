---
title: "GCP data"
order: 6
description: "Attacking GCP data services: Cloud SQL, BigQuery, Vertex AI, Dataproc, Dataflow, Firestore, Spanner, and Bigtable."
keywords:
  - Cloud SQL
  - BigQuery
  - Vertex AI
  - Dataproc
  - Firestore
  - data
---

# Data

The managed data stores are where a GCP engagement pays off, and reaching them splits into two moves that recur across every service here. Either the **data plane** is exposed directly (a public IP, an authorized network that is too wide, an over-broad `roles/*.dataViewer` binding), or a **service runs under an attached service account** that you can drive, so the job becomes both compute and a path to that account's token. The analytics and ML services in particular double as compute and are cross-referenced into [identity](../identity/index.md) where the service account is the prize.

## What folds in here

- **[Cloud SQL](cloud-sql.md)**: the SQL Auth Proxy, public IP and authorized networks, and built-in database users.
- **[BigQuery](bigquery.md)**: reading datasets, query and export exfiltration, and dataset IAM.
- **[Vertex AI](vertex-ai.md)**: notebook and training-job service accounts, and model and dataset theft.
- **[Dataproc](dataproc.md)**: the cluster service account, job submission, and staged data.
- **[Dataflow](dataflow.md)**: the worker service account and pipeline source and sink data.
- **[Firestore](firestore.md)**: collections through broad database IAM or security-rule gaps.
- **[Spanner](spanner.md)**: databases through `spanner.databases` access.
- **[Bigtable](bigtable.md)**: tables through `bigtable.tables` access.

## References

- [HackTricks Cloud: GCP databases](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Datadog Security Labs: cloud attack research](https://securitylabs.datadoghq.com/)
- [Google Cloud: IAM for data services](https://cloud.google.com/iam/docs/)
