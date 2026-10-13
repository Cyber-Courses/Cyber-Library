---
title: "AKS"
order: 2
description: "Attaquer Azure Kubernetes Service depuis le plan cloud : identifiants admin, command invoke et identité managée du node pool via IMDS."
keywords:
  - AKS
  - listClusterAdminCredential
  - command invoke
  - node pool
  - managed identity
---

# AKS

Azure Kubernetes Service fait le pont entre le plan de management Azure et l'API Kubernetes, si bien qu'un point d'ancrage Azure se transforme en pouvoir dans le cluster de deux façons : récupérer le kubeconfig admin du cluster, ou exécuter des commandes dans le cluster via le plan de management. Par ailleurs, chaque node pool porte une **identité managée** qu'un pod peut atteindre via IMDS.

## Pages

- **[Accès au cluster](cluster-access.md)** : `listClusterAdminCredential` et `managedClusters` `runCommand`.
- **[Identité des nœuds](node-identity.md)** : le kubelet du node pool et l'identité managée via IMDS.

Les attaques génériques dans le cluster Kubernetes (RBAC, jetons de compte de service, évasion de pod) vivent dans la section Conteneurs ; ces pages restent sur l'angle plan cloud AKS et identité des nœuds.

## Références

- [HackTricks Cloud : AKS](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : accès au cluster et identité AKS](https://learn.microsoft.com/azure/aks/concepts-identity)
- [Peirates (outil d'attaque Kubernetes)](https://github.com/inguardians/peirates)
