---
title: "API server enumeration: testing what an identity can do"
description: "With a service-account token, an attacker queries the Kubernetes API server to map permissions and resources. SelfSubjectAccessReview and SelfSubjectRulesReview reveal exactly what the identity may do, and listing secrets, pods, roles, and nodes exposes the cluster's contents and the escalation paths available to that token."
keywords:
  - kubernetes api
  - kubectl auth can-i
  - selfsubjectrulesreview
  - rbac enumeration
  - secrets
---

# API server enumeration

The API server is the cluster's single control point, and a service-account token authenticates to it. Enumeration here answers two questions: what can this identity do, and what is in the cluster. Kubernetes provides self-review endpoints that answer the first precisely, so there is no need to guess by trial; the second is ordinary `list`/`get` across the resources the token may read.

Point a client at the in-cluster API with the token:

```bash
APISERVER=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT
T=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
C=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
alias kapi="curl -s --cacert $C -H \"Authorization: Bearer $T\" $APISERVER"
```

## Map permissions precisely

```bash
# enumerate everything this identity may do (one call)
kubectl auth can-i --list                              # uses SelfSubjectRulesReview
# or via the API directly:
kapi -XPOST /apis/authorization.k8s.io/v1/selfsubjectrulesreviews \
  -H 'Content-Type: application/json' \
  -d '{"spec":{"namespace":"default"}}'
# test a specific dangerous permission
kubectl auth can-i create pods -n kube-system
kubectl auth can-i get secrets --all-namespaces
```

## Inventory the cluster

```bash
kapi /api/v1/secrets                                   # cluster secrets if permitted
kapi /api/v1/namespaces/<ns>/pods
kapi /apis/rbac.authorization.k8s.io/v1/clusterrolebindings
kapi /api/v1/nodes                                     # node names and addresses
```

## Exploitation notes

- `auth can-i --list` (SelfSubjectRulesReview) is the single most useful call: it returns the identity's full permission set, which immediately shows whether escalation verbs (`create pods`, `escalate`, `bind`, `impersonate`, `get secrets`) are available.
- Reading `secrets` is the direct win when permitted; otherwise the rules list points at which escalation page applies, for example [Over-permissive roles](../rbac-privilege-escalation/over-permissive-roles.md) or [Pod creation to node](../rbac-privilege-escalation/pod-creation-to-node.md).
- Anonymous or overly broad defaults sometimes let unauthenticated calls succeed; see [Anonymous API access](../exposed-components/anonymous-api-access.md).

## Tools

- [kubectl](https://kubernetes.io/docs/reference/kubectl/)
- [kubectl-who-can / rakkess (permission mapping)](https://github.com/corneliusweig/rakkess)

## References

- [Kubernetes: authorization overview](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)
- [Kubernetes: SelfSubjectRulesReview](https://kubernetes.io/docs/reference/kubernetes-api/authorization-resources/self-subject-rules-review-v1/)
