---
title: "Réseau : abuser du réseau du cluster, des politiques et du mesh"
order: 7
description: "Le réseau de Kubernetes offre plusieurs surfaces offensives : le réseau plat par défaut sans isolation, les NetworkPolicies qui peuvent être contournées ou absentes, les implémentations CNI et overlay avec leurs propres faiblesses, et les service meshes dont les sidecars et les plans de contrôle peuvent être détournés pour intercepter ou mal router le trafic et pour contourner les contrôles que le mesh est censé imposer."
keywords:
  - kubernetes network
  - networkpolicy
  - cni overlay
  - service mesh
  - traffic interception
---

# Réseau

Le réseau du cluster est à la fois une surface de mouvement et une surface de contrôle. Par défaut, le réseau est plat et non segmenté, de sorte que tout pod atteint n'importe quel autre ; là où des NetworkPolicies existent, elles sont souvent incomplètes ou contournables ; le CNI et l'overlay qui implémentent le réseau des pods ont leurs propres comportements abusables ; et un service mesh, censé ajouter authentification et chiffrement, introduit des sidecars et un plan de contrôle qui peuvent être retournés contre le trafic qu'ils gèrent.

```bash
ip -4 addr; ip route                                   # pod network and reachability
kubectl get networkpolicies -A 2>/dev/null             # are any policies defined at all?
kubectl get pods -A -o jsonpath='{..image}' | tr ' ' '\n' | grep -iE 'istio|linkerd|envoy|cilium' | sort -u
```

## Sous-sujets

- **[Contournement de NetworkPolicy](networkpolicy-bypass.md)** : atteindre des cibles qu'une politique devait bloquer.
- **[Abus du CNI et de l'overlay](cni-and-overlay-abuse.md)** : faiblesses dans l'implémentation du réseau de pods.
- **[Abus du service mesh](service-mesh-abuse.md)** : retourner les sidecars et le plan de contrôle du mesh contre le trafic.

## Références

- [Kubernetes : network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes : réseau du cluster](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
