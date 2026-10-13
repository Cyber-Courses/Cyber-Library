---
title: "Énumération du cluster : cartographier un cluster Kubernetes depuis un point d'ancrage"
order: 1
description: "Les premiers mouvements après avoir atterri dans un pod Kubernetes : inventorier ce que le pod détient déjà (son jeton de compte de service, les secrets montés, l'environnement), découvrir l'identité du compte de service et ce qu'elle peut faire contre le serveur d'API, énumérer les services et le réseau des pods, et atteindre le point de terminaison de métadonnées cloud pour les identifiants du nœud."
keywords:
  - énumération kubernetes
  - jeton de compte de service
  - serveur d'api
  - reconnaissance de cluster
  - pod
---

# Énumération du cluster

Un point d'ancrage dans Kubernetes est presque toujours un shell dans un pod. Le pod n'est pas un conteneur nu : on lui a donné une identité et une vue du cluster, et énumérer les deux est ce qui transforme le point d'ancrage en mouvement. L'ordre qui porte ses fruits consiste à inventorier d'abord le pod lui-même (le jeton de compte de service et les éventuels secrets montés suffisent fréquemment à agir immédiatement), puis à tester ce que cette identité peut faire contre le serveur d'API, puis à cartographier les services et le réseau, et enfin à atteindre le point de terminaison de métadonnées cloud pour les identifiants propres du nœud.

Les deux points de terminaison que chaque pod peut voir :

```bash
# the in-cluster API server, from injected environment
echo "$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT"      # usually 10.96.0.1:443
# the pod's own service-account token, CA, and namespace
ls /var/run/secrets/kubernetes.io/serviceaccount/            # token  ca.crt  namespace
```

## Sous-thèmes

- **[Énumération intra-pod](in-pod-enumeration.md)** : secrets, jetons et environnement à l'intérieur du pod.
- **[Découverte de comptes de service et de jetons](service-account-and-token-discovery.md)** : l'identité du pod et les autres jetons accessibles.
- **[Énumération du serveur d'API](api-server-enumeration.md)** : ce que l'identité peut lire et faire contre l'API.
- **[Découverte des services et du réseau](service-and-network-discovery.md)** : services, endpoints, DNS et réseau des pods.
- **[Métadonnées cloud depuis un pod](cloud-metadata-from-pod.md)** : atteindre les identifiants d'instance cloud du nœud.

## Références

- [Concepts de l'API Kubernetes](https://kubernetes.io/docs/reference/using-api/api-concepts/)
- [HackTricks : Kubernetes enumeration](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-enumeration)
- [NCC Group : Kubernetes security](https://research.nccgroup.com/)
