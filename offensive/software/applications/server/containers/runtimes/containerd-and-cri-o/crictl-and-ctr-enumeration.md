---
title: "crictl and ctr enumeration: inventorying containers and images on a node"
description: "Using crictl, ctr, or nerdctl against a reachable runtime socket to inventory the pods, containers, and images on a node, inspect their mounts and environment for secrets, and exec into running workloads before escalating."
keywords:
  - crictl
  - ctr
  - nerdctl
  - node enumeration
  - container inventory
---

# crictl and ctr enumeration

Before creating anything, read the node. The runtime tools list every pod, container, and image on the node and expose their configuration, which reveals mounts, environment secrets, and which workloads are worth targeting or exec-ing into.

```bash
crictl ps -a                                  # all containers on the node
crictl inspect <id> | jq '.info.config.envs, .info.runtimeSpec.mounts'
crictl exec -it <id> sh                       # shell into a running container
ctr -n k8s.io images list
```

## Exploitation notes

- Container `envs` and `mounts` frequently hold service-account tokens, database passwords, and cloud credentials.
- `crictl exec` into a more privileged workload is a lateral step that needs only socket access, not the API.
- Enumeration is quiet and read-only; use it to choose a target before the noisier create-and-escape step.

## References

- [crictl user guide](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
- [containerd documentation](https://containerd.io/docs/)
