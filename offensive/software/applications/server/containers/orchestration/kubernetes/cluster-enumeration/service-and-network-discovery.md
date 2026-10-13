---
title: "Service and network discovery: mapping services, endpoints, and the pod network"
order: 5
description: "Kubernetes gives every pod cluster DNS and a flat pod network by default. An attacker resolves service names, lists services and endpoints through the API or DNS, and scans the pod and service CIDRs to find reachable workloads, databases, and internal APIs that are frequently unauthenticated on the assumption that only cluster peers can reach them."
keywords:
  - kubernetes dns
  - services endpoints
  - pod network
  - cluster.local
  - network scanning
---

# Service and network discovery

By default a pod can resolve cluster DNS and reach every other pod and service on a flat network, because Kubernetes applies no network isolation unless a NetworkPolicy is present. That makes service and network discovery highly productive: internal services assume only in-cluster peers can reach them and are often unauthenticated, so simply finding and connecting to them yields access to databases, caches, message queues, and internal APIs.

## Discover services through the API and DNS

```bash
# services and their cluster IPs/ports via the API
kapi /api/v1/services | python3 -c 'import sys,json;[print(f"{s[\"metadata\"][\"namespace\"]}/{s[\"metadata\"][\"name\"]} {s[\"spec\"].get(\"clusterIP\")} {[p.get(\"port\") for p in s[\"spec\"].get(\"ports\",[])]}") for s in json.load(sys.stdin)["items"]]'
# endpoints map services to the actual pod IPs behind them
kapi /api/v1/endpoints
# cluster DNS: SRV records enumerate ports; names follow a fixed scheme
nslookup -type=srv _https._tcp.kubernetes.default.svc.cluster.local 2>/dev/null
getent hosts <service>.<namespace>.svc.cluster.local
```

Service DNS names are deterministic (`<service>.<namespace>.svc.cluster.local`), so even without API read access, guessing common names (`postgres`, `redis`, `elasticsearch`, `vault`) and resolving them finds backends.

## Scan the pod and service networks

```bash
# find the pod CIDR from the local address and routes
ip -4 addr; ip route
# sweep the pod network for reachable services (adjust the range)
for i in $(seq 1 254); do (nc -z -w1 10.244.0.$i 6379 2>/dev/null && echo "10.244.0.$i:6379 open") & done; wait
# common internal ports: 6379 redis, 5432 postgres, 9200 elasticsearch, 8080 apis
```

## Exploitation notes

- Endpoints are more useful than services for lateral movement because they list the real pod IPs; hitting a pod IP directly bypasses a service's own access assumptions.
- Internal services are frequently unauthenticated by design; a reachable Redis, Elasticsearch, or internal HTTP API often needs no credentials, making network discovery a direct data-access win.
- Where a NetworkPolicy does block traffic, see [NetworkPolicy bypass](../network/networkpolicy-bypass.md); the flat-network assumption holds only until a policy exists.
- Feed discovered reachable pods into [Pod-to-pod pivoting](../lateral-movement/pod-to-pod-pivoting.md).

## Tools

- [kube-hunter (cluster network recon)](https://github.com/aquasecurity/kube-hunter)
- [nmap](https://nmap.org/)

## References

- [Kubernetes: DNS for services and pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Kubernetes: services and endpoints](https://kubernetes.io/docs/concepts/services-networking/service/)
