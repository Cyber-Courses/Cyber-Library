---
title: "Instance metadata"
description: "Reading the GCP metadata server (metadata.google.internal) for the attached service account's token and SSH keys, including via SSRF to the metadata endpoint."
keywords:
  - instance metadata
  - metadata server
  - service account token
  - SSRF
  - GCE
  - computeMetadata
---

# Instance metadata

Every Compute Engine instance (and GKE node, Cloud Run, and Cloud Build worker) can reach the **metadata server** at `metadata.google.internal` (`169.254.169.254`). It serves the attached service account's OAuth token, its granted scopes, project metadata, and SSH keys, with no authentication beyond a required request header. On a foothold it is the first credential source; reached through a web vulnerability it is a server-side request forgery payload.

## What folds in here

- **[Service account token](service-account-token.md)**: the OAuth token and scopes from the service-accounts endpoint, on a shell.
- **[SSRF to metadata](ssrf-to-metadata.md)**: the same endpoint reached through a hosted-app request forgery.

## The required header

Every metadata request needs `Metadata-Flavor: Google`; requests without it are refused, which is what makes a naive SSRF that cannot set headers harder (and what a `?alt=json` and recursive read exploit once it can).

```bash
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/?recursive=true&alt=json'
```

## References

- [Google: VM metadata](https://cloud.google.com/compute/docs/metadata/overview)
- [Hacking the Cloud: GCP metadata SSRF](https://hackingthe.cloud/)
- [HackTricks Cloud: GCP metadata](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
