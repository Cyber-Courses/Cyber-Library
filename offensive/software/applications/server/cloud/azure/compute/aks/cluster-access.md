---
title: "Cluster access: admin kubeconfig and command invoke"
description: "Pulling AKS admin credentials with listClusterAdminCredential or running in-cluster commands with managedClusters runCommand."
keywords:
  - AKS
  - cluster admin credential
  - runCommand
  - kubeconfig
  - cluster access
---

# Cluster access

Two management-plane actions hand you the cluster. `listClusterAdminCredential` returns a **cluster-admin kubeconfig** that bypasses Azure RBAC entirely, and `managedClusters runCommand` runs `kubectl` (or any command) inside the cluster from the Azure side, which works even when the API server is private.

## Pull the admin kubeconfig

```bash
# admin creds ignore Azure AD / RBAC integration
az aks get-credentials -g <rg> -n <cluster> --admin
kubectl get secrets -A
```

## Run commands through the management plane

```bash
az aks command invoke -g <rg> -n <cluster> \
  -c "kubectl get secrets -A -o json"
```

## Exploitation notes

- `--admin` requires the `listClusterAdminCredential` action; it is the single most valuable AKS right because it sidesteps any Azure-AD cluster RBAC.
- `command invoke` reaches a **private** cluster with no network line of sight, since it runs through the Azure control plane.
- Cluster secrets routinely include further cloud credentials and service-account tokens for lateral movement.

## Tools

- **Azure CLI** (`az aks get-credentials --admin`, `az aks command invoke`).
- **kubectl** / **Peirates**: in-cluster enumeration and movement once you hold the kubeconfig.

## References

- [Microsoft: AKS cluster access](https://learn.microsoft.com/azure/aks/control-kubeconfig-access)
- [HackTricks Cloud: AKS](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Peirates](https://github.com/inguardians/peirates)
