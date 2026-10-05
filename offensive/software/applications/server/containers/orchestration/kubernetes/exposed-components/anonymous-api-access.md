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

The API server enables anonymous authentication by default, mapping unauthenticated requests to the user `system:anonymous` and the group `system:unauthenticated`. That group is bound by default to `system:public-info-viewer`, so anonymous callers can already read non-sensitive endpoints (version, health, discovery); that is expected, not a flaw. The danger is a binding that grants anonymous access to workload resources such as pods or secrets, a common mistake copied from tutorials.

```bash
API=https://<apiserver>:6443
curl -sk $API/version                                   # expected to work (public-info-viewer)
curl -sk $API/api/v1/namespaces/default/pods            # reading THIS anonymously is the flaw
curl -sk $API/api/v1/secrets                            # secrets without a token is critical
```

## Exploitation notes

- A `2xx` on a workload resource (pods, secrets) without a token means a dangerous anonymous binding; version, health, and discovery succeeding is the default and benign.
- The highest-impact misconfiguration binds anonymous to a role that can read secrets or create pods, which is immediate escalation.
- Distinct from this is the legacy [Insecure apiserver port](insecure-apiserver-port.md), which bypasses authentication entirely rather than mapping to an identity.

## References

- [Kubernetes: anonymous requests](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests)
- [Kubernetes: controlling access to the API](https://kubernetes.io/docs/concepts/security/controlling-access/)
