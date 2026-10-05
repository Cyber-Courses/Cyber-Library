---
title: "Supply chain: compromising what the cluster installs and trusts"
description: "Clusters install software through Helm charts and operators, both of which run with elevated rights and pull manifests and images an attacker may influence. A malicious or tampered chart, or an operator and its custom resources, executes attacker-defined objects with the installer's permissions, turning the cluster's own provisioning tooling into an execution and persistence path."
keywords:
  - kubernetes supply chain
  - helm chart
  - operator
  - custom resource
  - crd
---

# Supply chain

Kubernetes clusters install and manage software through tooling that runs with high privilege: Helm renders charts into cluster objects, and operators watch custom resources and reconcile them into workloads. Both consume definitions, charts, manifests, and images, that an attacker may be able to influence, and both act with the installer's or operator's permissions rather than the submitter's. A tampered chart or a malicious custom resource therefore executes attacker-defined objects at elevated privilege, making the provisioning layer an execution and persistence surface distinct from direct API abuse.

```bash
helm list -A 2>/dev/null                               # installed releases
kubectl get crd 2>/dev/null | head                     # operators' custom resources
kubectl get pods -A | grep -iE 'operator|controller'   # operator controllers and their SAs
```

## Subtopics

- **[Helm chart abuse](helm-chart-abuse.md)**: malicious or tampered charts executing at install time.
- **[Operator and CRD abuse](operator-and-crd-abuse.md)**: driving an operator through its custom resources.

## References

- [Helm: security considerations](https://helm.sh/docs/topics/provenance/)
- [Operator Framework](https://operatorframework.io/)
- [SLSA: supply-chain threats](https://slsa.dev/spec/v1.0/threats)
