---
title: "GKE"
description: "Attacking Google Kubernetes Engine from the cloud side: cluster credential retrieval, node service-account tokens, and Workload Identity."
keywords:
  - GKE
  - container.clusters.getCredentials
  - node identity
  - Workload Identity
  - kubelet
---

# GKE

Google Kubernetes Engine sits on the seam between Cloud IAM and Kubernetes RBAC, and both ends are attackable from the cloud side. A Cloud IAM permission (`container.clusters.getCredentials`) hands you a kubeconfig for the cluster; a shell on a node gives you the node pool's service-account token from the metadata server. The pages here cover that cloud-to-cluster and node-identity angle.

## What folds in here

- **[Cluster access](cluster-access.md)**: `container.clusters.getCredentials` and the get-credentials flow to reach the Kubernetes API.
- **[Node identity](node-identity.md)**: stealing the node pool's service-account token from metadata, and Workload Identity abuse.

Generic in-cluster Kubernetes attacks (RBAC, service-account tokens, pod escape) live in the Containers area; these pages stay on the GCP control-plane and node-identity angle.

## References

- [HackTricks Cloud: GCP GKE](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: GKE Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)
