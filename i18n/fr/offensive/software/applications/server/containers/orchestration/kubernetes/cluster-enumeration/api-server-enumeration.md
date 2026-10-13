---
title: "Énumération du serveur d'API : tester ce qu'une identité peut faire"
order: 2
description: "Avec un jeton de compte de service, un attaquant interroge le serveur d'API Kubernetes pour cartographier les permissions et les ressources. SelfSubjectAccessReview et SelfSubjectRulesReview révèlent exactement ce que l'identité peut faire, et lister les secrets, pods, rôles et nœuds expose le contenu du cluster et les voies d'escalade disponibles pour ce jeton."
keywords:
  - api kubernetes
  - kubectl auth can-i
  - selfsubjectrulesreview
  - énumération rbac
  - secrets
---

# Énumération du serveur d'API

Le serveur d'API est le point de contrôle unique du cluster, et un jeton de compte de service s'y authentifie. L'énumération répond ici à deux questions : que peut faire cette identité, et qu'y a-t-il dans le cluster. Kubernetes fournit des points de terminaison d'auto-examen qui répondent précisément à la première, de sorte qu'il n'est pas nécessaire de deviner par essais ; la seconde est un ordinaire `list`/`get` à travers les ressources que le jeton peut lire.

Pointez un client vers l'API intra-cluster avec le jeton :

```bash
APISERVER=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT
T=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
C=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
alias kapi="curl -s --cacert $C -H \"Authorization: Bearer $T\" $APISERVER"
```

## Cartographier les permissions avec précision

```bash
# enumerate everything this identity may do (one call)
kubectl auth can-i --list                              # uses SelfSubjectRulesReview
# or via the API directly:
kapi -XPOST /apis/authorization.k8s.io/v1/selfsubjectrulesreviews \
  -H 'Content-Type: application/json' \
  -d '{"spec":{"namespace":"default"}}'
# test a specific dangerous permission
kubectl auth can-i create pods -n kube-system
kubectl auth can-i get secrets --all-namespaces
```

## Inventorier le cluster

```bash
kapi /api/v1/secrets                                   # cluster secrets if permitted
kapi /api/v1/namespaces/<ns>/pods
kapi /apis/rbac.authorization.k8s.io/v1/clusterrolebindings
kapi /api/v1/nodes                                     # node names and addresses
```

## Notes d'exploitation

- `auth can-i --list` (SelfSubjectRulesReview) est l'appel le plus utile à lui seul : il renvoie l'ensemble complet des permissions de l'identité, ce qui montre immédiatement si des verbes d'escalade (`create pods`, `escalate`, `bind`, `impersonate`, `get secrets`) sont disponibles.
- Lire les `secrets` est le gain direct lorsqu'il est permis ; sinon, la liste des règles indique quelle page d'escalade s'applique, par exemple [Rôles trop permissifs](../rbac-privilege-escalation/over-permissive-roles.md) ou [Création de pod vers nœud](../rbac-privilege-escalation/pod-creation-to-node.md).
- Des valeurs par défaut anonymes ou trop larges laissent parfois réussir des appels non authentifiés ; voir [Accès API anonyme](../exposed-components/anonymous-api-access.md).

## Outils

- [kubectl](https://kubernetes.io/docs/reference/kubectl/)
- [kubectl-who-can / rakkess (cartographie des permissions)](https://github.com/corneliusweig/rakkess)

## Références

- [Kubernetes : vue d'ensemble de l'autorisation](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)
- [Kubernetes : SelfSubjectRulesReview](https://kubernetes.io/docs/reference/kubernetes-api/authorization-resources/self-subject-rules-review-v1/)
