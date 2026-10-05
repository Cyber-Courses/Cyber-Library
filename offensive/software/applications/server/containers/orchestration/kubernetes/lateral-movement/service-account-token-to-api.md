---
title: "Service account token to API: authenticating with a stolen token"
description: "Using a stolen Kubernetes service-account token to authenticate to the API server and act as that identity, enumerating and exercising its permissions across namespaces to move laterally and escalate within the cluster."
keywords:
  - service account token
  - API authentication
  - bearer token
  - lateral movement
  - kubernetes identity
---

# Service account token to API

A service-account token is a bearer credential for the API. With one in hand, point a client at the API server and act as that account. The first move is to learn what it can do, then exercise it: read more secrets, create pods, or reach namespaces the original foothold could not.

```bash
API=https://kubernetes.default.svc
kubectl --server=$API --token=<stolen> --insecure-skip-tls-verify auth can-i --list
kubectl --server=$API --token=<stolen> --insecure-skip-tls-verify get secrets -A
```

## Exploitation notes

- Tokens are namespace-bound identities but their RBAC can be cluster-wide; `auth can-i --list` reveals the true scope.
- Projected tokens are audience-bound and short-lived; use them promptly and against the intended API audience.
- Chain into [RBAC privilege escalation](../rbac-privilege-escalation/index.md) if the token can escalate, bind, impersonate, or create pods.

## References

- [Kubernetes: authenticating](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
- [Kubernetes: service account tokens](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
