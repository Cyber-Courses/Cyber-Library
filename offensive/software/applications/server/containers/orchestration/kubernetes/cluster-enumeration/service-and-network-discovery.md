---
title: "Service and network discovery: mapping the cluster network from a pod"
description: "Mapping a Kubernetes cluster's services and pod network from a foothold: using in-cluster DNS to enumerate services, scanning the flat pod and service CIDRs for reachable endpoints, and identifying node and control-plane addresses to target next."
keywords:
  - kubernetes network
  - kube-dns
  - service discovery
  - pod CIDR
  - cluster scanning
---

# Service and network discovery

Kubernetes pod networks are usually flat: a pod can reach most services and other pods directly. Combined with in-cluster DNS, that makes service and endpoint discovery straightforward and productive.

```bash
# In-cluster DNS resolves services; the API and kubelet are well-known
nslookup kubernetes.default.svc.cluster.local
getent hosts kube-dns.kube-system.svc.cluster.local

# Scan the flat pod/service network for reachable endpoints
for ip in 10.0.0.{1..254}; do (nc -z -w1 $ip 10250 2>/dev/null && echo "$ip kubelet") & done; wait
```

## Exploitation notes

- Service DNS names (`<svc>.<ns>.svc.cluster.local`) enumerate the application and its dependencies without touching the API.
- The kubelet (`10250`), etcd (`2379`), and dashboards are high-value endpoints to look for on the node and control-plane addresses.
- A flat network is the precondition for [Pod to pod pivoting](../lateral-movement/pod-to-pod-pivoting.md); where a NetworkPolicy exists, see [NetworkPolicy bypass](../network/networkpolicy-bypass.md).

## References

- [Kubernetes: DNS for services and pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Kubernetes: cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
