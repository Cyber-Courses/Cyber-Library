---
title: "Anonymous API access: an unauthenticated Kubernetes API server"
description: "Reaching a Kubernetes API server that allows anonymous requests, where unauthenticated callers are bound to system:anonymous, and testing what that identity can do, since a misconfigured anonymous binding can read secrets or create workloads."
keywords:
  - anonymous API
  - system:anonymous
  - unauthenticated kubernetes
  - API server
  - RBAC misconfiguration
---

# Anonymous API access

The API server enables anonymous authentication by default, mapping unauthenticated requests to the user `system:anonymous` and the group `system:unauthenticated`. That is safe only while no role is bound to those principals. A binding that grants them anything, a common mistake copied from tutorials, exposes the API without credentials.

```bash
API=https://<apiserver>:6443
curl -sk $API/version                                   # reachable?
curl -sk $API/api/v1/namespaces/default/pods            # anonymous read?
curl -sk $API/apis/rbac.authorization.k8s.io/v1/clusterrolebindings | \
  grep -i 'system:anonymous\|system:unauthenticated'
```

## Exploitation notes

- Any `2xx` on a resource without a token means an anonymous binding exists; enumerate what it grants with the same unauthenticated calls.
- The highest-impact misconfiguration binds anonymous to a role that can read secrets or create pods, which is immediate escalation.
- Distinct from this is the legacy [Insecure apiserver port](insecure-apiserver-port.md), which bypasses authentication entirely rather than mapping to an identity.

## References

- [Kubernetes: anonymous requests](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests)
- [Kubernetes: controlling access to the API](https://kubernetes.io/docs/concepts/security/controlling-access/)
