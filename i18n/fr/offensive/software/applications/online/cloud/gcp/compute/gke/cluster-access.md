---
title: "Accès au cluster : get-credentials vers l'API Kubernetes"
description: "Récupérer des identifiants cluster-admin GKE avec container.clusters.getCredentials et le flux get-credentials pour atteindre l'API Kubernetes."
keywords:
  - GKE
  - getCredentials
  - cluster admin
  - Kubernetes API
  - kubeconfig
---

# Accès au cluster

GKE s'authentifie auprès de l'API Kubernetes avec un jeton Cloud IAM : la permission Cloud IAM `container.clusters.get` (via `container.clusters.getCredentials`) suffit donc à obtenir un kubeconfig fonctionnel. Ce que vous pouvez ensuite faire à l'intérieur du cluster dépend du RBAC Kubernetes lié à votre identité, mais les rôles larges `roles/container.admin` ou `roles/container.clusterAdmin` correspondent directement à cluster-admin.

## Obtenir un kubeconfig

```bash
# list clusters you can reach, then pull credentials
gcloud container clusters list
gcloud container clusters get-credentials <cluster> --zone <zone> --project <project>

# now talk to the API with your GCP identity as the bearer
kubectl auth can-i --list
kubectl get secrets -A
```

## Notes d'exploitation

- `roles/container.admin` accorde cluster-admin Kubernetes via le RBAC ; même `container.viewer` combiné à `getCredentials` permet souvent de lire des secrets à travers les namespaces.
- Un cluster privé répond quand même si vous pouvez atteindre l'endpoint de son plan de contrôle (depuis une VM dans le VPC, un réseau peeré ou un réseau autorisé où vous avez atterri).
- Une fois à l'intérieur, pivotez vers les comptes de service de pods et l'identité du nœud ; les techniques génériques internes au cluster se trouvent dans la section Conteneurs.

## Outils

- **gcloud** (`container clusters get-credentials`).
- **kubectl**, **Peirates** pour le déplacement interne au cluster une fois le kubeconfig détenu.

## Références

- [HackTricks Cloud: GCP GKE](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: container.clusters.getCredentials](https://cloud.google.com/sdk/gcloud/reference/container/clusters/get-credentials)
