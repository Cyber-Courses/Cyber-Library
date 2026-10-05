---
title: "etcd: reading every cluster secret from the datastore"
description: "Reaching the etcd datastore that backs the Kubernetes API, directly or with recovered client certificates, to read every object in the cluster including all secrets and service-account tokens, or to write objects and bypass the API server entirely."
keywords:
  - etcd
  - 2379
  - cluster datastore
  - kubernetes secrets
  - etcdctl
---

# etcd

etcd stores the entire cluster state, and it stores Secrets unencrypted unless encryption-at-rest is configured. Reaching it, on `2379` without client-certificate enforcement or with recovered certs, reads every Secret object and all other stored state at once, and writing to it changes cluster state behind the API server's back. Projected service-account tokens are issued on demand through TokenRequest and are not stored here, so currently-mounted tokens come from pods or nodes rather than etcd.

```bash
export ETCDCTL_API=3
etcdctl --endpoints=https://<node>:2379 \
  --cert=client.crt --key=client.key --cacert=ca.crt \
  get / --prefix --keys-only | grep /secrets/

etcdctl ... get /registry/secrets/<ns>/<name>        # the secret's raw value
```

## Exploitation notes

- Every Secret object is here, including legacy service-account token secrets; a privileged one among them is cluster-admin. Short-lived projected tokens are not in etcd, so recover those through [Token and secret theft](../lateral-movement/token-and-secret-theft.md).
- Client certificates are found on control-plane nodes under `/etc/kubernetes/pki/etcd/`; recovering them is etcd access.
- Where encryption-at-rest is on, Secret values are ciphertext, but tokens and other objects are still exposed.

## References

- [Kubernetes: securing etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
- [Kubernetes: encrypting secrets at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
