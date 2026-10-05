---
title: "Compute Engine"
description: "Attacking Compute Engine VMs: metadata and SSH key injection, OS Login, and the instance's attached service-account scopes."
keywords:
  - Compute Engine
  - GCE
  - metadata
  - SSH
  - OS Login
---

# Compute Engine

A Compute Engine VM binds two things an attacker wants: a way to **run code** (through instance metadata, which the guest trusts for startup scripts and SSH keys) and an **identity** (the attached service account, whose token the metadata server dispenses). Holding `compute.instances.setMetadata` on an instance is effectively code execution on it; reaching the metadata server from a shell on it is its service account.

## What folds in here

- **[Metadata and SSH](metadata-and-ssh.md)**: writing a startup script or SSH key to land a shell, and reading the metadata server on-host.
- **[Service account scope](service-account-scope.md)**: using the instance's attached service account and OAuth scopes to call Google APIs.

Creating a *new* VM with a privileged service account attached is the [actAs compute-instance](../../identity/privilege-escalation/act-as-deployment/compute-instance.md) escalation; these pages cover existing instances.

## References

- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Google: Compute Engine metadata](https://cloud.google.com/compute/docs/metadata/overview)
