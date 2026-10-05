---
title: "RBAC privilege escalation: turning limited Kubernetes rights into more"
description: "Escalating privileges inside Kubernetes RBAC: abusing the escalate and bind verbs, impersonation, over-permissive roles and wildcards, the right to create pods, certificate signing request approval, and the TokenRequest API to obtain a stronger identity up to cluster-admin."
keywords:
  - kubernetes RBAC
  - privilege escalation
  - escalate bind verbs
  - impersonation
  - service account token
---

# RBAC privilege escalation

Kubernetes authorization is RBAC, and several verbs and resources let a modest identity become a stronger one. Some are explicit escalation primitives the API guards (escalate, bind, impersonate); others are powerful by second-order effect (create pods, approve certificates, request tokens). Map your rights with `auth can-i --list`, then take the shortest path up.

## Subtopics

- **[Over-permissive roles](over-permissive-roles.md)**: wildcards and dangerous verbs granted outright.
- **[Escalate and bind verbs](escalate-and-bind-verbs.md)**: granting yourself more through RBAC itself.
- **[Impersonation](impersonation.md)**: acting as another, more privileged identity.
- **[Service account token abuse](service-account-token-abuse.md)**: reading and reusing other accounts' tokens.
- **[Pod creation to node](pod-creation-to-node.md)**: turning pod creation into node and cluster compromise.
- **[CSR approval](csr-approval.md)**: minting a client certificate for a privileged identity.
- **[TokenRequest API](tokenrequest-api.md)**: minting tokens for service accounts you can act on.

## References

- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Kubernetes: privilege escalation prevention](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#privilege-escalation-prevention-and-bootstrapping)
