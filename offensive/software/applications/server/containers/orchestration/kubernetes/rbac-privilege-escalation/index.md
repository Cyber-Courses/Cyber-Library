---
title: "RBAC privilege escalation: turning a weak identity into cluster admin"
order: 3
description: "Kubernetes RBAC grants verbs on resources, and several verb-and-resource combinations let a limited identity grant itself more: over-permissive roles, the escalate and bind verbs, impersonation, certificate signing, the TokenRequest API, and the ability to create pods. Each converts a modest service account into broad or full cluster control."
keywords:
  - kubernetes rbac
  - privilege escalation
  - escalate bind
  - impersonate
  - service account
---

# RBAC privilege escalation

Kubernetes authorises actions as verbs (`get`, `list`, `create`, `update`) on resources (`pods`, `secrets`, `roles`). RBAC is meant to prevent an identity from granting itself more than it has, but several specific permissions break that containment. Holding any one of them turns a limited service account into a path to cluster admin: a role that is too broad, the `escalate` or `bind` verbs that defeat the anti-escalation check, `impersonate` to become another identity, certificate signing to mint a privileged client cert, the TokenRequest API to mint other service accounts' tokens, or the ability to create pods and schedule a node-owning one.

Map the identity's permissions first:

```bash
kubectl auth can-i --list                           # the full rule set for this identity
# flag the dangerous ones:
kubectl auth can-i create pods --all-namespaces
kubectl auth can-i get secrets --all-namespaces
kubectl auth can-i impersonate users
kubectl auth can-i create clusterrolebindings
kubectl auth can-i '*' '*'
```

## Subtopics

- **[Over-permissive roles](over-permissive-roles.md)**: wildcard or secret-reading roles.
- **[Escalate and bind verbs](escalate-and-bind-verbs.md)**: granting yourself more than you hold.
- **[Impersonation](impersonation.md)**: acting as another user, group, or service account.
- **[CSR approval](csr-approval.md)**: minting a privileged client certificate.
- **[TokenRequest API](tokenrequest-api.md)**: minting tokens for other service accounts.
- **[Service account token abuse](service-account-token-abuse.md)**: mounting or reusing powerful tokens.
- **[Pod creation to node](pod-creation-to-node.md)**: scheduling a pod that owns its node.

## References

- [Kubernetes: RBAC authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Kubernetes: privilege escalation prevention](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#privilege-escalation-prevention-and-bootstrapping)
- [HackTricks: Kubernetes RBAC](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
