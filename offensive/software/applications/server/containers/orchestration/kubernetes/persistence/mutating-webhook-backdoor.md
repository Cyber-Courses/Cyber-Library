---
title: "Mutating webhook backdoor: injecting into every new workload"
description: "Persisting in Kubernetes with a mutating admission webhook that modifies objects as they are created, injecting a sidecar, a volume, or an entrypoint into every new pod cluster-wide, so the attacker's code rides along with legitimate workloads automatically."
keywords:
  - mutating webhook
  - sidecar injection
  - admission control
  - kubernetes persistence
  - workload tampering
---

# Mutating webhook backdoor

A mutating admission webhook rewrites objects before they are stored. Pointed at an attacker endpoint and scoped to pods, it injects into every new workload: an extra sidecar container, a hostPath volume, or a modified command. The injection is automatic and cluster-wide, so legitimate deployments carry the attacker's code.

```bash
# Registration (requires admissionregistration write): a MutatingWebhookConfiguration
# matching pods, pointed at the attacker's webhook service, that returns a JSONPatch
# adding a privileged sidecar to spec.containers.
kubectl get mutatingwebhookconfigurations
```

## Exploitation notes

- Injecting a privileged or hostPath sidecar re-establishes node access on every new pod, so the foothold regenerates faster than defenders remove it.
- Scope the webhook narrowly (namespaces, labels) to limit noise while still covering high-value workloads.
- It needs write to `mutatingwebhookconfigurations` and a reachable webhook endpoint; a benign configuration name helps it blend in.

## References

- [Kubernetes: dynamic admission control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [Kubernetes: admission webhook good practices](https://kubernetes.io/docs/concepts/cluster-administration/admission-webhooks-good-practices/)
