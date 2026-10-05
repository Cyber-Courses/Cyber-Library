---
title: "Persistence: keeping access to a Kubernetes cluster"
description: "Persisting in a Kubernetes cluster after compromise: running attacker workloads as deployments and cronjobs, hiding static pods on nodes, backdooring RBAC with hidden roles and bindings, and registering admission webhooks that reinfect or re-privilege every new workload."
keywords:
  - kubernetes persistence
  - malicious workload
  - static pod
  - RBAC backdoor
  - admission webhook
---

# Persistence

Persistence in Kubernetes hides in the cluster's own mechanisms: controllers that keep workloads running, cronjobs that fire on a schedule, static pods the API never sees, RBAC objects that quietly restore access, and admission webhooks that touch every new object. The best footholds look like ordinary cluster configuration.

## Subtopics

- **[Malicious workloads](malicious-workloads.md)**: deployments and daemonsets that self-heal.
- **[CronJobs](cronjobs.md)**: scheduled attacker execution.
- **[Static pods](static-pods.md)**: node-level pods outside the API.
- **[RBAC backdoor](rbac-backdoor.md)**: hidden roles, bindings, and accounts.
- **[Admission webhooks](admission-webhooks.md)**: intercepting cluster operations.
- **[Mutating webhook backdoor](mutating-webhook-backdoor.md)**: injecting into every new workload.

## References

- [Kubernetes: controllers](https://kubernetes.io/docs/concepts/architecture/controller/)
- [MITRE ATT&CK: Containers matrix](https://attack.mitre.org/matrices/enterprise/containers/)
