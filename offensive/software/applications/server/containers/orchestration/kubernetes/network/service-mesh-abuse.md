---
title: "Service mesh abuse: bypassing and attacking a service mesh"
description: "Bypassing or abusing a Kubernetes service mesh such as Istio or Linkerd: escaping the sidecar to send traffic that skips mesh policy and mTLS, exploiting permissive mTLS modes, and attacking the mesh control plane and its broad identity."
keywords:
  - service mesh
  - Istio
  - Linkerd
  - sidecar bypass
  - mTLS
---

# Service mesh abuse

A service mesh routes pod traffic through a sidecar proxy that enforces mTLS and policy. Its guarantees hold only while traffic actually goes through the sidecar and while mTLS is required. Sending traffic that bypasses the sidecar, exploiting permissive mTLS, or compromising the control plane defeats the mesh.

```bash
# Sidecar interception relies on iptables redirection the pod may be able to alter or avoid
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].name}' | tr ' ' '\n' | grep -i 'istio-proxy\|linkerd-proxy'
# Direct pod-IP traffic can skip mesh policy where the proxy is not in path
```

## Exploitation notes

- Permissive mTLS modes accept plaintext, so an attacker who reaches a pod IP directly bypasses the mesh's authentication.
- Traffic to ports or destinations the sidecar does not capture escapes policy; excluded ranges and host-network pods are common gaps.
- The mesh control plane holds a powerful identity and can reconfigure routing cluster-wide; it is a high-value target in its own right.

## References

- [Istio security](https://istio.io/latest/docs/concepts/security/)
- [Linkerd security](https://linkerd.io/2/features/automatic-mtls/)
