---
title: "Cluster access: get-credentials to the Kubernetes API"
description: "Retrieving GKE cluster-admin credentials with container.clusters.getCredentials and the get-credentials flow to reach the Kubernetes API."
keywords:
  - GKE
  - getCredentials
  - cluster admin
  - Kubernetes API
  - kubeconfig
---

# Cluster access

GKE authenticates to the Kubernetes API with a Cloud IAM token, so the Cloud IAM permission `container.clusters.get` (via `container.clusters.getCredentials`) is enough to obtain a working kubeconfig. What you can then do inside the cluster depends on the Kubernetes RBAC bound to your identity, but the broad `roles/container.admin` or `roles/container.clusterAdmin` map straight to cluster-admin.

## Getting a kubeconfig

```bash
# list clusters you can reach, then pull credentials
gcloud container clusters list
gcloud container clusters get-credentials <cluster> --zone <zone> --project <project>

# now talk to the API with your GCP identity as the bearer
kubectl auth can-i --list
kubectl get secrets -A
```

## Exploitation notes

- `roles/container.admin` grants Kubernetes cluster-admin through RBAC; even `container.viewer` plus `getCredentials` often reads secrets across namespaces.
- A private cluster still answers if you can reach its control-plane endpoint (from a VM in the VPC, a peered network, or an authorized network you landed in).
- Once inside, pivot to pod service accounts and the node identity; generic in-cluster technique lives in the Containers area.

## Tools

- **gcloud** (`container clusters get-credentials`).
- **kubectl**, **Peirates** for in-cluster movement once you hold the kubeconfig.

## References

- [HackTricks Cloud: GCP GKE](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: container.clusters.getCredentials](https://cloud.google.com/sdk/gcloud/reference/container/clusters/get-credentials)
