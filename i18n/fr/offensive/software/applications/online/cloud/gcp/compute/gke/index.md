---
title: "GKE"
description: "Attaquer Google Kubernetes Engine depuis le côté cloud : récupération des identifiants de cluster, jetons de compte de service de nœud et Workload Identity."
keywords:
  - GKE
  - container.clusters.getCredentials
  - node identity
  - Workload Identity
  - kubelet
---

# GKE

Google Kubernetes Engine se situe à la couture entre Cloud IAM et Kubernetes RBAC, et les deux extrémités sont attaquables depuis le côté cloud. Une permission Cloud IAM (`container.clusters.getCredentials`) vous remet un kubeconfig pour le cluster ; un shell sur un nœud vous donne le jeton du compte de service du node pool depuis le serveur de métadonnées. Les pages ici couvrent cet angle cloud-vers-cluster et identité-de-nœud.

## Ce qui se rattache ici

- **[Accès au cluster](cluster-access.md)** : `container.clusters.getCredentials` et le flux get-credentials pour atteindre l'API Kubernetes.
- **[Identité du nœud](node-identity.md)** : voler le jeton du compte de service du node pool depuis les métadonnées, et abuser de Workload Identity.

Les attaques Kubernetes génériques internes au cluster (RBAC, jetons de compte de service, évasion de pod) se trouvent dans la section Conteneurs ; ces pages restent sur l'angle plan-de-contrôle GCP et identité-de-nœud.

## Références

- [HackTricks Cloud: GCP GKE](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: GKE Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)
