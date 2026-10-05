---
title: "Pod to pod pivoting: reaching other workloads over the pod network"
description: "Moving laterally across a Kubernetes cluster's flat pod network from one compromised pod to others, reaching their exposed services and APIs directly, since by default any pod can connect to any other pod and service in the cluster."
keywords:
  - pod to pod
  - flat network
  - lateral movement
  - service access
  - kubernetes pivot
---

# Pod to pod pivoting

By default Kubernetes gives every pod reach to every other pod and service: the network is flat unless a NetworkPolicy restricts it. From one compromised pod, the others are directly reachable, so their services, admin interfaces, and unauthenticated endpoints become targets.

```bash
# Enumerate services and endpoints, then connect directly
kubectl get svc,endpoints -A 2>/dev/null
for ip in $(kubectl get pods -A -o jsonpath='{.items[*].status.podIP}'); do
  nc -z -w1 $ip 6379 2>/dev/null && echo "$ip redis"; done
```

## Exploitation notes

- Internal services frequently skip authentication because they assume the network is trusted; a flat network breaks that assumption.
- Target databases, caches, message queues, and internal admin UIs reachable from the pod network.
- Where a NetworkPolicy is present, see [NetworkPolicy bypass](../network/networkpolicy-bypass.md); discovery is covered in [Service and network discovery](../cluster-enumeration/service-and-network-discovery.md).

## References

- [Kubernetes: network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes: cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
