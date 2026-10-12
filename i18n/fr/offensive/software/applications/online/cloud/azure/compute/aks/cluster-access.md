---
title: "Accès au cluster : kubeconfig admin et command invoke"
description: "Récupération des identifiants admin AKS avec listClusterAdminCredential ou exécution de commandes dans le cluster avec managedClusters runCommand."
keywords:
  - AKS
  - cluster admin credential
  - runCommand
  - kubeconfig
  - cluster access
---

# Accès au cluster

Deux actions du plan de management vous livrent le cluster. `listClusterAdminCredential` renvoie un **kubeconfig cluster-admin** qui contourne entièrement le RBAC Azure, et `managedClusters runCommand` exécute `kubectl` (ou n'importe quelle commande) à l'intérieur du cluster depuis le côté Azure, ce qui fonctionne même lorsque l'API server est privé.

## Récupérer le kubeconfig admin

```bash
# admin creds ignore Azure AD / RBAC integration
az aks get-credentials -g <rg> -n <cluster> --admin
kubectl get secrets -A
```

## Exécuter des commandes via le plan de management

```bash
az aks command invoke -g <rg> -n <cluster> \
  -c "kubectl get secrets -A -o json"
```

## Notes d'exploitation

- `--admin` nécessite l'action `listClusterAdminCredential` ; c'est le droit AKS le plus précieux, car il contourne tout RBAC de cluster basé sur Azure AD.
- `command invoke` atteint un cluster **privé** sans aucune visibilité réseau, puisqu'il passe par le plan de contrôle Azure.
- Les secrets du cluster incluent régulièrement d'autres identifiants cloud et des jetons de compte de service pour le déplacement latéral.

## Outils

- **Azure CLI** (`az aks get-credentials --admin`, `az aks command invoke`).
- **kubectl** / **Peirates** : énumération et déplacement dans le cluster une fois le kubeconfig obtenu.

## Références

- [Microsoft : accès au cluster AKS](https://learn.microsoft.com/azure/aks/control-kubeconfig-access)
- [HackTricks Cloud : AKS](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Peirates](https://github.com/inguardians/peirates)
