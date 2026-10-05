---
title: "cAdvisor and metrics: harvesting container and node telemetry"
description: "Scraping cAdvisor, metrics-server, and node metrics endpoints that are exposed without authentication to inventory containers, processes, and resource usage across nodes, and to recover command lines and labels that leak secrets and map high-value workloads."
keywords:
  - cAdvisor
  - metrics-server
  - 4194
  - kubernetes telemetry
  - reconnaissance
---

# cAdvisor and metrics

Telemetry endpoints are reconnaissance gold. cAdvisor (historically on `4194`, and proxied by the kubelet) and metrics endpoints expose per-container detail: images, labels, command lines, and resource usage across the node or cluster. They rarely allow code execution, but they map the environment and often leak secrets in process arguments.

```bash
curl -sk https://<node>:10250/metrics/cadvisor | head
curl -s http://<node>:4194/api/v1.3/subcontainers | jq '.[].spec.labels'   # legacy cAdvisor
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods                         # metrics-server
```

## Exploitation notes

- Container labels and command lines frequently embed tokens, connection strings, and flags that name the next target.
- The inventory identifies privileged and high-value pods to focus on for [Kubelet API](kubelet-api.md) exec or token theft.
- These endpoints are read-only; treat them as a quiet mapping step before acting.

## References

- [cAdvisor](https://github.com/google/cadvisor)
- [Kubernetes: resource metrics pipeline](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/)
