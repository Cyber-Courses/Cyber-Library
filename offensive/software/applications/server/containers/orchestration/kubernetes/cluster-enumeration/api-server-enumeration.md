---
title: "API server enumeration: probing the Kubernetes API with your identity"
description: "Enumerating the Kubernetes API server from a pod with a service-account token: listing namespaces, pods, secrets, roles, and bindings the identity can read, and reading the RBAC graph to find escalation paths and high-value resources."
keywords:
  - kubernetes API server
  - kubectl get
  - RBAC enumeration
  - secrets listing
  - cluster recon
---

# API server enumeration

With a token, read everything the identity allows. The API server exposes the full cluster state, and even a limited identity often reads secrets, roles, and bindings that chart the next move.

```bash
k get namespaces
k get pods -A -o wide
k get secrets -A                      # tokens, registry creds, app secrets
k get roles,rolebindings,clusterroles,clusterrolebindings -A
k get nodes -o wide
```

## Exploitation notes

- `get secrets` is the highest-value read: service-account tokens and application credentials live there, and one readable secret often grants a stronger identity.
- The roles and bindings are the escalation map; read them to find who can `escalate`, `impersonate`, or create pods, as in [RBAC privilege escalation](../rbac-privilege-escalation/index.md).
- Where direct reads are denied, `auth can-i --list` still reveals the shape of your permissions without triggering access failures on each resource.

## References

- [Kubernetes API concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
