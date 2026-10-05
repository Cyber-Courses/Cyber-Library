---
title: "Exposed kubeconfig: authenticating with a recovered credential file"
description: "Recovering a kubeconfig or client certificate from a host, container image, CI artifact, or developer machine, and using it to authenticate to the Kubernetes API as whatever identity it carries, frequently an administrator."
keywords:
  - kubeconfig
  - client certificate
  - credential theft
  - CI artifact
  - kubernetes authentication
---

# Exposed kubeconfig

A kubeconfig bundles the API endpoint and a credential, a client certificate, a token, or an exec plugin. They leak constantly: in home directories, baked into images, in CI secrets and logs, and on bastion hosts. A recovered kubeconfig is direct API access as its identity, which is often a cluster or namespace admin.

```bash
find / -name '*.kube/config' -o -name 'kubeconfig' 2>/dev/null
# Inspect what identity and cluster it holds, then use it
kubectl --kubeconfig ./found.config config view --minify
kubectl --kubeconfig ./found.config auth can-i --list
```

## Exploitation notes

- Developer and CI kubeconfigs frequently carry admin-level rights for convenience; `auth can-i --list` confirms the power.
- Client certificates in a kubeconfig cannot be revoked by rotation the way tokens can, so a leaked cert is durable access until the CA is rotated.
- Images and CI artifacts are prime hunting grounds; combine with [Secrets in image layers](../../../runtimes/docker/images-and-registries/secrets-in-image-layers.md).

## References

- [Kubernetes: organizing cluster access with kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
- [Kubernetes: PKI certificates](https://kubernetes.io/docs/setup/best-practices/certificates/)
