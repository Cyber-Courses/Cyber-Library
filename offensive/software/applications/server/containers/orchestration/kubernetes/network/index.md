---
title: "Network: abusing cluster networking, policies, and the mesh"
description: "Kubernetes networking offers several offensive surfaces: the flat default network with no isolation, NetworkPolicies that can be bypassed or are absent, CNI and overlay implementations with their own weaknesses, and service meshes whose sidecars and control planes can be abused to intercept or misroute traffic and to bypass the controls the mesh is supposed to enforce."
keywords:
  - kubernetes network
  - networkpolicy
  - cni overlay
  - service mesh
  - traffic interception
---

# Network

Cluster networking is both a movement surface and a control surface. By default the network is flat and unsegmented, so any pod reaches any other; where NetworkPolicies exist they are often incomplete or bypassable; the CNI and overlay that implement pod networking have their own abusable behaviours; and a service mesh, meant to add authentication and encryption, introduces sidecars and a control plane that can be turned against the traffic they manage.

```bash
ip -4 addr; ip route                                   # pod network and reachability
kubectl get networkpolicies -A 2>/dev/null             # are any policies defined at all?
kubectl get pods -A -o jsonpath='{..image}' | tr ' ' '\n' | grep -iE 'istio|linkerd|envoy|cilium' | sort -u
```

## Subtopics

- **[NetworkPolicy bypass](networkpolicy-bypass.md)**: reaching targets a policy was meant to block.
- **[CNI and overlay abuse](cni-and-overlay-abuse.md)**: weaknesses in the pod-network implementation.
- **[Service mesh abuse](service-mesh-abuse.md)**: turning sidecars and the mesh control plane against traffic.

## References

- [Kubernetes: network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes: cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
