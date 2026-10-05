---
title: "Kubernetes: attacking the cluster from a pod to full control"
description: "Attacking Kubernetes end to end: enumerating the cluster from a compromised pod, reaching exposed components, escalating through RBAC and service-account tokens, escaping a pod to its node, moving between workloads and into the cloud, and persisting in the cluster."
keywords:
  - kubernetes
  - k8s attack
  - RBAC
  - pod escape
  - service account token
---

# Kubernetes

Most Kubernetes attacks start from a single compromised pod and climb. A pod carries an identity (a service-account token), sits on a flat network, and runs on a node that holds credentials for the whole cluster. The path is familiar: enumerate, reach exposed components, escalate through RBAC, escape to the node, move laterally, and persist. Escaping the pod to its node uses the runtime-agnostic primitives in [Container escape](../../container-escape/index.md); this area is the cluster-level attack model around them.

## Subtopics

- **[Cluster enumeration](cluster-enumeration/index.md)**: mapping the cluster and your identity from a pod.
- **[Exposed components](exposed-components/index.md)**: unauthenticated or reachable control-plane and node components.
- **[RBAC privilege escalation](rbac-privilege-escalation/index.md)**: turning limited permissions into more.
- **[Pod escape to node](pod-escape-to-node/index.md)**: breaking out of a pod to its node.
- **[Lateral movement](lateral-movement/index.md)**: moving between workloads, namespaces, and into the cloud.
- **[Persistence](persistence/index.md)**: keeping access across restarts and remediation.
- **[Network](network/index.md)**: attacking the cluster network and service mesh.
- **[Supply chain](supply-chain/index.md)**: Helm, operators, and admission as deployment paths.

## References

- [Kubernetes security concepts](https://kubernetes.io/docs/concepts/security/)
- [MITRE ATT&CK: Containers matrix](https://attack.mitre.org/matrices/enterprise/containers/)
