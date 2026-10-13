---
title: "Chaîne d'approvisionnement : compromettre ce que le cluster installe et approuve"
order: 8
description: "Les clusters installent des logiciels via des charts Helm et des opérateurs, qui s'exécutent tous deux avec des droits élevés et récupèrent des manifestes et des images qu'un attaquant peut influencer. Un chart malveillant ou altéré, ou un opérateur et ses ressources personnalisées, exécute des objets définis par l'attaquant avec les permissions de l'installateur, transformant l'outillage de provisionnement du cluster en voie d'exécution et de persistance."
keywords:
  - kubernetes supply chain
  - helm chart
  - operator
  - custom resource
  - crd
---

# Chaîne d'approvisionnement

Les clusters Kubernetes installent et gèrent des logiciels via un outillage qui s'exécute avec des privilèges élevés : Helm rend les charts en objets de cluster, et les opérateurs surveillent les ressources personnalisées et les réconcilient en charges de travail. Tous deux consomment des définitions, des charts, des manifestes et des images qu'un attaquant peut être en mesure d'influencer, et tous deux agissent avec les permissions de l'installateur ou de l'opérateur plutôt que celles de l'émetteur. Un chart altéré ou une ressource personnalisée malveillante exécute donc des objets définis par l'attaquant à un niveau de privilège élevé, faisant de la couche de provisionnement une surface d'exécution et de persistance distincte de l'abus direct de l'API.

```bash
helm list -A 2>/dev/null                               # installed releases
kubectl get crd 2>/dev/null | head                     # operators' custom resources
kubectl get pods -A | grep -iE 'operator|controller'   # operator controllers and their SAs
```

## Sous-thèmes

- **[Abus de charts Helm](helm-chart-abuse.md)** : des charts malveillants ou altérés s'exécutant au moment de l'installation.
- **[Abus d'opérateurs et de CRD](operator-and-crd-abuse.md)** : piloter un opérateur via ses ressources personnalisées.

## Références

- [Helm : considérations de sécurité](https://helm.sh/docs/topics/provenance/)
- [Operator Framework](https://operatorframework.io/)
- [SLSA: supply-chain threats](https://slsa.dev/spec/v1.0/threats)
