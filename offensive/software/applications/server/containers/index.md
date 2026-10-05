---
title: "Containers: offensive techniques against runtimes and orchestration"
description: "Attacking containerized workloads across three layers: the runtime-agnostic escape from a container to its host, the container runtimes and the control planes they expose, and the orchestration layer that schedules containers across a fleet of hosts."
keywords:
  - container security
  - container escape
  - Docker
  - Kubernetes
  - container runtime
---

# Containers

Containers package a workload and run it as ordinary host processes that the kernel isolates with namespaces, cgroups, capabilities, and a seccomp or LSM profile. That isolation is a configuration, not a boundary the hardware enforces, so offensive work against containers is mostly about where the isolation is weak, missing, or handed back to the workload on purpose.

The subject splits into three layers, each with its own attack model.

## Subtopics

- **[Container escape](container-escape/index.md)**: breaking out of a single container to its host. These primitives are runtime-agnostic: a privileged flag, a dangerous mount, a shared host namespace, or a runtime or kernel exploit works the same from a Docker container, a Podman container, or a Kubernetes pod.

Two sibling layers complete the subject and are covered in their own areas: the **Runtimes** (the container engines and their control planes and image supply chain) and **Orchestration** (the Kubernetes cluster control plane).

## References

- [NCC Group: Understanding and Hardening Linux Containers](https://research.nccgroup.com/2016/04/13/understanding-and-hardening-linux-containers/)
- [Docker engine security](https://docs.docker.com/engine/security/)
- [Kubernetes security concepts](https://kubernetes.io/docs/concepts/security/)
