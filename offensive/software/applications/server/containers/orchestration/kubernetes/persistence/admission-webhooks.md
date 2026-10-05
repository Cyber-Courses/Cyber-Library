---
title: "Admission webhooks: intercepting cluster operations for persistence"
description: "Persisting in Kubernetes by registering an admission webhook that intercepts API requests, using a validating or mutating webhook as a cluster-wide hook to observe every operation, exfiltrate submitted secrets, or deny operations that would remove the attacker's access."
keywords:
  - admission webhook
  - validating webhook
  - dynamic admission
  - kubernetes persistence
  - interception
---

# Admission webhooks

Dynamic admission webhooks are called on API operations before objects are persisted. As persistence they are a cluster-wide hook: a webhook can observe every create and update (including the full object, so submitted secrets flow through it), and a validating webhook can deny operations, for example blocking attempts to delete the attacker's resources.

```bash
# A validating webhook pointed at an attacker-controlled endpoint sees matching operations
kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations
# Registration requires admissionregistration.k8s.io write rights
```

## Exploitation notes

- A webhook receiving `secrets` and `serviceaccounts/token` operations exfiltrates credentials as they are created, cluster-wide.
- A `failurePolicy: Ignore` keeps the cluster working if the endpoint is down, which hides the backdoor; `Fail` can be used to deny defender actions.
- To actively re-infect new workloads rather than only observe, use a [Mutating webhook backdoor](mutating-webhook-backdoor.md).

## References

- [Kubernetes: dynamic admission control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [Kubernetes: admission controllers reference](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
