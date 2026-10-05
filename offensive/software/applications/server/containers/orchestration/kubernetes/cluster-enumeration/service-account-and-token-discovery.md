---
title: "Service account and token discovery: finding and testing pod identities"
description: "Discovering the service-account token mounted into a pod and testing what it can do with self-subject access reviews, then hunting for additional, more privileged tokens in mounted secrets and on the node to pick the strongest identity available."
keywords:
  - service account token
  - SelfSubjectAccessReview
  - kubectl auth can-i
  - token discovery
  - kubernetes identity
---

# Service account and token discovery

The default service-account token is the first identity, but rarely the best. Test what it can do, then look for stronger tokens in other mounted secrets, in the kubelet's on-node secret store, and in environment or config.

```bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
APISERVER=https://kubernetes.default.svc
alias k="kubectl --server=$APISERVER --token=$TOKEN --certificate-authority=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"

# What can this identity do?
k auth can-i --list
k auth can-i create pods
```

## Exploitation notes

- `auth can-i --list` is the fastest map of a token's power; look for `create pods`, `get secrets`, `escalate`, `impersonate`, and wildcard verbs.
- More privileged tokens hide in mounted secrets of co-located pods and, after a node foothold, in `/var/lib/kubelet/pods/*/volumes/kubernetes.io~secret/`.
- Pick the strongest identity before acting; the next steps (RBAC escalation, pod creation) depend on it.

## References

- [Kubernetes: authorization overview](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)
- [Kubernetes: checking API access](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#checking-api-access)
