---
title: "Image pull secret theft: recovering registry credentials from the runtime"
description: "Recovering registry pull credentials from a container runtime's on-node configuration and state, including Kubernetes image pull secrets materialized on the node and the runtime's auth files, to authenticate to private registries and reach more images."
keywords:
  - image pull secret
  - registry credentials
  - kubelet secrets
  - docker config.json
  - node credential theft
---

# Image pull secret theft

To pull private images, the runtime stores registry credentials on the node, and Kubernetes materializes image pull secrets there too. A node foothold recovers them from the runtime's auth files and from the kubelet's on-disk secret state, then reuses them against the registry.

```bash
# Runtime and kubelet auth material on the node
find / -name config.json -path '*containers*' 2>/dev/null -exec cat {} \;
cat /var/lib/kubelet/config.json 2>/dev/null
ls /var/lib/kubelet/pods/*/volumes/kubernetes.io~secret/*/.dockerconfigjson 2>/dev/null
```

## Exploitation notes

- Pull credentials often reach registries beyond the images on this node, widening access to the whole project's private images.
- Recovered credentials feed [Registry access](../docker/images-and-registries/registry-access.md); push rights among them enable backdooring.
- Kubelet-materialized secrets on the node also include non-registry secrets mounted into pods, worth sweeping at the same time.

## References

- [Kubernetes: pull an image from a private registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
- [containerd documentation](https://containerd.io/docs/)
