---
title: "Contournement de NetworkPolicy : atteindre des cibles qu'une politique devait isoler"
order: 1
description: "Les NetworkPolicies sont des listes d'autorisation appliquées par le CNI, et elles échouent de façons prévisibles : les namespaces sans politique sont entièrement ouverts, les politiques omettent souvent l'egress ou ne couvrent que certains pods, le DNS et l'API server restent couramment accessibles, et les chemins au niveau du nœud ou du réseau de l'hôte contournent les règles à portée de pod. Un attaquant cartographie les lacunes et contourne l'isolation prévue."
keywords:
  - networkpolicy
  - egress
  - cni enforcement
  - dns
  - bypass
---

# Contournement de NetworkPolicy

Une NetworkPolicy est une liste d'autorisation du trafic permis pour les pods qu'elle sélectionne, appliquée par le CNI. Ses faiblesses sont structurelles plutôt que basées sur un exploit. Une politique ne s'applique qu'aux pods correspondant à son sélecteur, de sorte que les pods et les namespaces entiers sans politique sont sans restriction. Les politiques définissent fréquemment l'ingress mais omettent l'egress, ou inversement. Elles laissent couramment le DNS et l'API server accessibles parce que les charges de travail en ont besoin. Et les règles à portée du trafic des pods ne contraignent pas les chemins au niveau du nœud ou du réseau de l'hôte. Cartographier ces lacunes est la façon dont un attaquant atteint une cible que la politique était censée isoler.

Trouvez les lacunes :

```bash
# which namespaces and pods have no policy at all?
kubectl get networkpolicies -A
kubectl get ns -o name | while read ns; do \
  n=$(kubectl get netpol -n ${ns#namespace/} --no-headers 2>/dev/null | wc -l); \
  echo "${ns#namespace/}: $n policies"; done
# does a given policy cover egress, or only ingress?
kubectl get netpol -n <ns> -o yaml | grep -E 'policyTypes|Ingress|Egress'
```

## Contournements d'une politique

```bash
# 1. a namespace or pod with no policy is fully reachable; pivot through it
# 2. egress not restricted: exfiltrate and reach external or other-namespace targets
# 3. DNS left open: tunnel over DNS, or use allowed resolver paths
# 4. host-network path: a hostNetwork pod is on the node stack, outside pod-scoped rules
#    (see Host namespaces) so it reaches node-local and policy-exempt destinations
# 5. the CNI may not enforce policy on certain traffic (e.g. node->pod, or hairpin)
```

## Notes d'exploitation

- La première vérification est la couverture : tout namespace sans aucune politique est ouvert, et les clusters protègent couramment quelques namespaces sensibles en laissant le reste à plat, de sorte qu'un namespace voisin est souvent un pivot sans restriction.
- Les omissions d'egress sont la lacune fonctionnelle la plus courante ; une politique qui ne filtre que l'ingress laisse encore un pod compromis atteindre tout vers l'extérieur, y compris les services d'autres namespaces par IP.
- Un pod `hostNetwork` échappe entièrement à la politique à portée de pod car son trafic provient du nœud ; à combiner avec [Host namespaces](../pod-escape-to-node/host-namespaces.md).
- L'application des politiques dépend du fait que le CNI implémente réellement NetworkPolicy ; certaines configurations acceptent les objets mais ne les appliquent pas, ce que `kubectl get netpol` ne peut pas révéler, aussi testez l'accessibilité directement.

## Références

- [Kubernetes: network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes: network policy gotchas](https://kubernetes.io/docs/concepts/services-networking/network-policies/#what-you-can-t-do-with-network-policies-currently)
