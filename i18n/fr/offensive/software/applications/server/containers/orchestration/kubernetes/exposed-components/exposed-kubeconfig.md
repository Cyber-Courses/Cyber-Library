---
title: "kubeconfig exposé : identifiants du cluster laissés à portée d'un attaquant"
order: 9
description: "Un fichier kubeconfig contient l'adresse du serveur et les identifiants pour s'y authentifier, souvent un certificat client ou un jeton pour un utilisateur à forts privilèges. Ces fichiers fuitent dans les répertoires personnels, les variables de CI, les images de conteneurs, les dépôts et les sauvegardes. Un kubeconfig trouvé donne un accès direct au cluster, au niveau de privilège que porte l'identifiant intégré, fréquemment cluster-admin."
keywords:
  - kubeconfig
  - cluster credentials
  - client certificate
  - kubectl config
  - credential leak
---

# kubeconfig exposé

Un kubeconfig réunit tout ce qu'il faut pour atteindre un cluster : l'URL de l'API server, l'autorité de certification du cluster, et un identifiant, qui est couramment un certificat client ou un jeton bearer pour un utilisateur ou un compte de service donné. Les kubeconfig des administrateurs portent fréquemment cluster-admin. Ces fichiers sont largement copiés et mal protégés, apparaissant dans les répertoires personnels, les secrets et l'environnement de CI/CD, commités dans des dépôts, intégrés dans des images, et dans des sauvegardes. En trouver un donne un accès direct au cluster au niveau de privilège de l'identifiant.

Chasser les kubeconfig :

```bash
# standard and common locations
cat ~/.kube/config 2>/dev/null
find / -name '*.kubeconfig' -o -name 'config' -path '*/.kube/*' 2>/dev/null
find / -name 'admin.conf' -o -name 'kubelet.conf' 2>/dev/null   # on control-plane nodes
# in CI/env and images
env | grep -iE 'KUBECONFIG|KUBE_'
grep -rIl 'apiVersion: v1' / 2>/dev/null | xargs grep -l 'clusters:' 2>/dev/null | head
```

## Utiliser l'identifiant

```bash
export KUBECONFIG=/path/to/found/config
kubectl config view --minify                          # server, user, and credential type
kubectl auth can-i --list                             # the privilege it carries
kubectl get secrets --all-namespaces                  # if admin, read everything
```

## Notes d'exploitation

- `admin.conf` sur un nœud du plan de contrôle est un certificat client cluster-admin ; c'est le kubeconfig de plus grande valeur et il est accessible après toute compromission d'un nœud du plan de contrôle.
- Le type d'identifiant intégré compte : un certificat client est durable et n'est pas révoqué par la suppression d'une liaison, tandis qu'un jeton peut être à durée de vie courte ; `kubectl config view` montre celui dont vous disposez.
- Les kubeconfig dans l'historique Git, les couches d'image et les secrets de CI sont fréquents ; fouillez les couches d'image ([Secrets dans les couches d'image](../../../runtimes/docker/images-and-registries/secrets-in-image-layers.md)) et les environnements de CI pour les y trouver.
- Exécutez toujours `auth can-i --list` en premier pour connaître le privilège avant d'agir, plutôt que de présumer un accès admin.

## Références

- [Kubernetes : organiser l'accès au cluster avec kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
- [HackTricks : Kubernetes kubeconfig](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
