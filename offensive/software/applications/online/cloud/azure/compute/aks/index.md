---
title: "AKS"
order: 2
description: "Attacking Azure Kubernetes Service from the cloud plane: admin credentials, command invoke, and node-pool managed identity through IMDS."
keywords:
  - AKS
  - listClusterAdminCredential
  - command invoke
  - node pool
  - managed identity
---

# AKS

Azure Kubernetes Service bridges the Azure management plane and the Kubernetes API, so an Azure foothold becomes in-cluster power two ways: pull the cluster's admin kubeconfig, or run commands in the cluster through the management plane. Separately, each node pool carries a **managed identity** that a pod can reach through IMDS.

## Pages

- **[Cluster access](cluster-access.md)**: `listClusterAdminCredential` and `managedClusters` `runCommand`.
- **[Node identity](node-identity.md)**: the node pool kubelet and managed identity through IMDS.

Generic in-cluster Kubernetes attacks (RBAC, service-account tokens, pod escape) live in the Containers area; these pages stay on the AKS cloud-plane and node-identity angle.

## References

- [HackTricks Cloud: AKS](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: AKS cluster access and identity](https://learn.microsoft.com/azure/aks/concepts-identity)
- [Peirates (Kubernetes attack tool)](https://github.com/inguardians/peirates)
