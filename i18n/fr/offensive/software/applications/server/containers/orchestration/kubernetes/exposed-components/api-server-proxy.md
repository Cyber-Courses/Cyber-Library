---
title: "Proxy du serveur d'API : atteindre les nœuds et les services via le plan de contrôle"
order: 7
description: "Le serveur d'API peut relayer des requêtes vers les nœuds, les pods et les services via ses sous-ressources proxy. Une identité autorisée à les utiliser atteint le kubelet de n'importe quel nœud et n'importe quel service ClusterIP, y compris ceux internes et non authentifiés, en utilisant le serveur d'API comme pivot qui contourne la segmentation réseau entre l'attaquant et ces cibles."
keywords:
  - proxy du serveur d'api
  - nodes proxy
  - services proxy
  - kubelet
  - pivot
---

# Proxy du serveur d'API

Le serveur d'API expose des sous-ressources proxy qui transfèrent une requête vers un nœud, un pod ou un service : `nodes/proxy`, `pods/proxy` et `services/proxy`. Leur usage prévu est l'introspection du cluster, mais pour un attaquant ce sont un pivot. Le serveur d'API se trouve sur le réseau du cluster et peut atteindre le kubelet de chaque nœud et chaque service ClusterIP, de sorte que relayer à travers lui contourne la segmentation qui sépare l'attaquant de ces cibles, et atteint les services internes qui supposent que seuls des appelants intra-cluster peuvent se connecter.

Vérifiez la permission et utilisez le proxy :

```bash
kubectl auth can-i get nodes/proxy
kubectl auth can-i get services/proxy
# proxy to a node's kubelet through the API server
kapi /api/v1/nodes/<node>/proxy/pods
kapi /api/v1/nodes/<node>/proxy/runningpods/
# proxy to an internal ClusterIP service (even if network-isolated from you)
kapi /api/v1/namespaces/<ns>/services/<scheme>:<svc>:<port>/proxy/
```

## Atteindre le kubelet pour l'exécution

Le proxy de nœud sert de façade au kubelet, de sorte que là où le kubelet expose des points de terminaison run/exec, relayer vers eux exécute des commandes dans les pods de ce nœud via le serveur d'API :

```bash
# proxy an exec/run to the kubelet (effect depends on kubelet config)
kapi -XPOST "/api/v1/nodes/<node>/proxy/run/<ns>/<pod>/<container>" -d 'cmd=id'
```

## Notes d'exploitation

- `nodes/proxy` est l'octroi à forte valeur : il sert de façade à l'API du kubelet sur n'importe quel nœud, de sorte qu'il se combine avec les voies de l'[API Kubelet](kubelet-api.md) pour lister les pods et s'y exécuter à l'échelle du cluster.
- `services/proxy` atteint les services internes et non authentifiés indépendamment de toute NetworkPolicy entre vous et eux, car la connexion provient du serveur d'API ; utilisez-le pour atteindre des tableaux de bord, des bases de données et des API internes.
- Le proxy conserve l'identité API de l'attaquant pour l'autorisation au niveau du serveur d'API, mais la cible relayée (un kubelet, un service interne) applique sa propre authentification, souvent plus faible.

## Références

- [Kubernetes : sous-ressources proxy](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/node-v1/#proxy)
- [Kubernetes : construire manuellement des URL de proxy apiserver](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster-services/)
- [HackTricks : Kubernetes API proxy](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
