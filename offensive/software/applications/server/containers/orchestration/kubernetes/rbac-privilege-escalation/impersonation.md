---
title: "Impersonation: acting as a more privileged Kubernetes identity"
description: "Escalating in Kubernetes with impersonation rights, which let an identity send requests as another user, group, or service account, so an attacker granted impersonate can act as an administrator or a privileged group without holding those permissions directly."
keywords:
  - kubernetes impersonation
  - impersonate verb
  - as-user
  - group impersonation
  - privilege escalation
---

# Impersonation

Impersonation lets a caller run a request as someone else by setting impersonation headers, which `kubectl` exposes as `--as` and `--as-group`. An identity granted the `impersonate` verb on users, groups, or service accounts can borrow their permissions, and impersonating a privileged group like `system:masters` is cluster-admin.

```bash
kubectl auth can-i impersonate users
kubectl auth can-i impersonate groups

# Act as a privileged group or admin user
kubectl --as=admin get secrets -A
kubectl --as=null --as-group=system:masters get nodes
```

## Exploitation notes

- Impersonating the group `system:masters` grants full cluster-admin, since that group is hard-wired to it.
- Impersonation can be scoped to specific names; where it is, enumerate which users, groups, or service accounts you may impersonate.
- It leaves the request attributed to the impersonated identity, which is also why it is powerful: you act fully as them.

## References

- [Kubernetes: user impersonation](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)
- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
