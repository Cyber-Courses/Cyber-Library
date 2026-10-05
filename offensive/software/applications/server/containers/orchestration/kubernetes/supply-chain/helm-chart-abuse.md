---
title: "Helm chart abuse: executing attacker objects through a chart"
description: "A Helm chart is a template that renders into arbitrary cluster objects, applied with the installing identity's permissions. An attacker who can get a malicious or tampered chart installed, through a poisoned repository, a crafted chart, or hooks, has the chart create privileged pods, RBAC bindings, or backdoors at install time, inheriting the installer's rights rather than their own."
keywords:
  - helm chart
  - helm hooks
  - chart repository
  - supply chain
  - privileged objects
---

# Helm chart abuse

Helm renders a chart's templates into Kubernetes objects and applies them with the credentials of whoever runs the install, which in CI or GitOps is frequently a powerful service account. Nothing constrains what those objects are, so a malicious chart simply includes the objects an attacker wants: a privileged pod, a ClusterRoleBinding to `cluster-admin`, a backdoor DaemonSet. Delivery is through a poisoned or typosquatted chart repository, a tampered chart in a trusted repo, or chart hooks that run jobs at defined lifecycle points. Because the objects are created with the installer's rights, the attacker inherits those rights without holding them.

```bash
helm repo list; helm search repo <name>                # configured repositories
# inspect a chart's rendered objects before/without installing
helm template ./chart | grep -iE 'ClusterRoleBinding|privileged|hostPath|hook'
```

## Malicious chart content

```yaml
# templates/backdoor.yaml rendered and applied with the installer's permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata: { name: {{ .Release.Name }}-metrics }
roleRef: { apiGroup: rbac.authorization.k8s.io, kind: ClusterRole, name: cluster-admin }
subjects:
- { kind: ServiceAccount, name: default, namespace: {{ .Release.Namespace }} }
```

```yaml
# a pre-install hook runs a Job before the rest, useful for one-shot actions
annotations:
  "helm.sh/hook": pre-install
# the Job's pod can be privileged with a host mount, giving node access at install
```

## Exploitation notes

- The privilege comes from the installer, not the chart author, so the target is any pipeline or operator that installs charts with a strong identity; get your content into a chart it installs.
- Hooks (`pre-install`, `post-install`) run Jobs at defined points and are an easy place to hide a one-shot privileged action that is less visible than a standing object.
- Delivery mirrors the image supply chain: a poisoned or typosquatted repo, or a tampered chart in a trusted one; see [Base image poisoning](../../../runtimes/docker/images-and-registries/base-image-poisoning.md) for the analogous image route.
- Inspect with `helm template` to see exactly what a chart renders before trusting it; attacker objects hide among legitimate ones.

## References

- [Helm: charts and hooks](https://helm.sh/docs/topics/charts_hooks/)
- [Helm: provenance and integrity](https://helm.sh/docs/topics/provenance/)
- [SLSA: supply-chain threats](https://slsa.dev/spec/v1.0/threats)
