---
title: "Supply chain: attacking what gets deployed into a cluster"
description: "Attacking the Kubernetes deployment supply chain: malicious or over-privileged Helm charts and tampered chart repositories, and abusing operators and the custom resources they reconcile to make a broadly privileged controller act on the attacker's behalf."
keywords:
  - kubernetes supply chain
  - helm chart
  - operator
  - custom resource
  - CRD
---

# Supply chain

Clusters deploy third-party software through charts and operators, which run with broad permissions and are trusted to create workloads and bindings. Attacking that path, a poisoned chart or an abused operator, deploys attacker intent through a trusted, privileged channel.

## Subtopics

- **[Helm chart abuse](helm-chart-abuse.md)**: malicious or over-privileged charts.
- **[Operator and CRD abuse](operator-and-crd-abuse.md)**: turning an operator's permissions against the cluster.

## References

- [Helm security](https://helm.sh/docs/topics/securing_installation/)
- [Kubernetes: operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
