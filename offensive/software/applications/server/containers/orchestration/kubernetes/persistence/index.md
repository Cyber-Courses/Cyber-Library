---
title: "Persistence: holding access to a compromised cluster"
description: "After gaining control, an attacker keeps it by planting mechanisms that survive credential rotation and pod restarts: RBAC bindings granting a durable identity, malicious workloads and cronjobs that re-establish access, static pods the kubelet runs directly, and admission webhooks that inject backdoors or mint credentials on every relevant API request."
keywords:
  - kubernetes persistence
  - rbac backdoor
  - static pod
  - admission webhook
  - cronjob
---

# Persistence

Holding a Kubernetes cluster means surviving the obvious responses: a rotated token, a deleted pod, a patched node. The durable mechanisms plant themselves in cluster state or on nodes so they outlast those. An RBAC binding grants a long-lived identity that no token rotation removes; a workload or cronjob re-creates access on a schedule; a static pod runs from a node's manifest directory with no API object to delete; and an admission webhook sits in the request path, injecting backdoors or minting credentials whenever a matching object is created.

```bash
# what can the current identity create for persistence?
kubectl auth can-i create clusterrolebindings
kubectl auth can-i create mutatingwebhookconfigurations
kubectl auth can-i create cronjobs -A
```

## Subtopics

- **[RBAC backdoor](rbac-backdoor.md)**: a durable identity through bindings.
- **[Malicious workloads](malicious-workloads.md)**: deployments and daemonsets that re-establish access.
- **[CronJobs](cronjobs.md)**: scheduled re-entry.
- **[Static pods](static-pods.md)**: pods the kubelet runs from a node manifest directory.
- **[Admission webhooks](admission-webhooks.md)**: intercepting the API request path.
- **[Mutating webhook backdoor](mutating-webhook-backdoor.md)**: injecting into every matching object.

## References

- [Kubernetes: RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [MITRE ATT&CK: persistence (containers)](https://attack.mitre.org/tactics/TA0003/)
- [Microsoft: threat matrix for Kubernetes](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
