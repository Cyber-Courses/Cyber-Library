---
title: "GCP compute"
description: "Attacking GCP compute: Compute Engine metadata, SSH, and service-account scopes, and GKE cluster access and node identity."
keywords:
  - Compute Engine
  - GKE
  - metadata
  - service account scope
  - SSH
---

# Compute

GCP compute is where a project's **service accounts** are most exposed: every Compute Engine VM and GKE node runs as a service account whose token the metadata server hands out on request, and the permission to write a VM's metadata is the permission to run code on it. The surfaces below turn control of, or reach to, an instance into that instance's identity.

## What folds in here

- **[Compute Engine](compute-engine/index.md)**: metadata and SSH-key injection, OS Login, and the instance's attached service-account scopes.
- **[GKE](gke/index.md)**: pulling cluster credentials from the cloud side, and stealing the node pool's service-account token.

Deploying a *new* instance to run as a privileged service account is a privilege-escalation path and lives under [identity actAs](../identity/privilege-escalation/act-as-deployment/compute-instance.md); the pages here cover instances and clusters that already exist. Generic in-cluster Kubernetes attacks live in the Containers area and are cross-referenced, not duplicated.

## References

- [HackTricks Cloud: GCP compute](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
