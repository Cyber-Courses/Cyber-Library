---
title: "RBAC backdoor: hidden roles, bindings, and accounts"
description: "Persisting in Kubernetes by planting RBAC objects that quietly restore access: a service account with a broad binding, a role binding to a built-in group, or permissions attached to a default account, so the attacker keeps a path back even after the initial foothold is removed."
keywords:
  - RBAC backdoor
  - clusterrolebinding
  - service account
  - hidden permissions
  - kubernetes persistence
---

# RBAC backdoor

RBAC objects are quiet persistence. A new service account bound to `cluster-admin`, a binding that grants a built-in group broad rights, or extra permissions attached to an existing default account all survive pod cleanup and look like ordinary cluster configuration. The backdoored account and its binding are the durable path back; a token is minted from them whenever needed.

```bash
kubectl create serviceaccount metrics-agent -n kube-system
kubectl create clusterrolebinding metrics-agent --clusterrole=cluster-admin \
  --serviceaccount=kube-system:metrics-agent
kubectl create token metrics-agent -n kube-system       # short-lived token; re-mint from the durable SA
```

## Exploitation notes

- Binding to an existing, innocuously named service account in `kube-system` is less conspicuous than an obvious `admin` account.
- Granting a built-in group like `system:authenticated` a dangerous permission backdoors every authenticated identity at once.
- This requires the RBAC-write rights obtained through [RBAC privilege escalation](../rbac-privilege-escalation/index.md).

## References

- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Kubernetes: service accounts](https://kubernetes.io/docs/concepts/security/service-accounts/)
