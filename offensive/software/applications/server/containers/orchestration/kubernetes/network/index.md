---
title: "Network: attacking the Kubernetes cluster network"
description: "Attacking the Kubernetes network layer: bypassing NetworkPolicy isolation, abusing the CNI plugin and overlay to spoof identities and sniff cross-node traffic, and attacking the service mesh through sidecar bypass, mTLS gaps, and the mesh control plane."
keywords:
  - kubernetes network
  - NetworkPolicy
  - CNI
  - service mesh
  - overlay network
---

# Network

The cluster network is flat and trusting by default, and the controls layered on top (NetworkPolicy, CNI features, a service mesh) each have gaps. Attacking this layer defeats the isolation defenders rely on to segment workloads and to protect internal services.

## Subtopics

- **[NetworkPolicy bypass](networkpolicy-bypass.md)**: reaching pods a policy was meant to isolate.
- **[CNI and overlay abuse](cni-and-overlay-abuse.md)**: spoofing and sniffing on the pod network.
- **[Service mesh abuse](service-mesh-abuse.md)**: bypassing or abusing a mesh.

## References

- [Kubernetes: network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes: cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
