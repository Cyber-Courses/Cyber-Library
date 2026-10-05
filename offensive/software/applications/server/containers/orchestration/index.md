---
title: "Orchestration: attacking the container cluster control plane"
description: "Attacking the orchestration layer that schedules and manages containers across a fleet of hosts. Kubernetes is the dominant target: enumerating the cluster, reaching exposed components, escalating through RBAC, escaping a pod to its node, moving laterally, and persisting."
keywords:
  - container orchestration
  - kubernetes
  - cluster control plane
  - k8s security
  - orchestration attack
---

# Orchestration

Orchestration schedules containers across many hosts and gives them a shared identity, network, and secret store. That control plane is the attack surface: compromising it reaches every workload and node at once. Kubernetes is the overwhelming target, so this layer is organized around it, from first foothold in a pod through cluster takeover and persistence.

## Subtopics

- **[Kubernetes](kubernetes/index.md)**: the dominant orchestrator, attacked from a pod foothold up to full cluster control.

## References

- [Kubernetes security concepts](https://kubernetes.io/docs/concepts/security/)
- [MITRE ATT&CK: Containers matrix](https://attack.mitre.org/matrices/enterprise/containers/)
