---
title: "Kubernetes: attacking the cluster from a pod to full control"
description: "Attacking Kubernetes end to end: enumerating the cluster from a compromised pod, reaching exposed components, escalating through RBAC and service-account tokens, escaping a pod to its node, moving between workloads and into the cloud, and persisting in the cluster."
keywords:
  - kubernetes
  - k8s attack
  - RBAC
  - pod escape
  - service account token
---

# Kubernetes

Most Kubernetes attacks start from a single compromised pod and climb. A pod carries an identity (a service-account token), sits on a flat network, and runs on a node that holds credentials for the whole cluster. The path is familiar: enumerate, reach exposed components, escalate through RBAC, escape to the node, move laterally, and persist. Escaping the pod to its node uses the runtime-agnostic primitives in [Container escape](../../container-escape/index.md); this area is the cluster-level attack model around them.

## First moves from a pod

These few commands decide which branch of the area applies next:

```bash
# 1. identity and what it can do (drives the RBAC and lateral branches)
T=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
echo "$T" | cut -d. -f2 | base64 -d 2>/dev/null | python3 -m json.tool   # whoami
kubectl --token="$T" --certificate-authority=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  --server=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT auth can-i --list

# 2. am I in an over-privileged pod? (drives pod-escape-to-node)
grep -E 'CapEff|Seccomp' /proc/self/status; cat /proc/self/uid_map
mount | grep -vE 'overlay|proc|sysfs|tmpfs|cgroup' | head   # hostPath

# 3. what is reachable? (drives exposed-components, network, cloud)
curl -sk https://$KUBERNETES_SERVICE_HOST/version
curl -s http://169.254.169.254/latest/meta-data/ 2>/dev/null | head   # cloud metadata
```

Read the results: broad `can-i` output points at [RBAC privilege escalation](rbac-privilege-escalation/index.md); a privileged or host-mounting pod points at [Pod escape to node](pod-escape-to-node/index.md); a reachable kubelet, etcd, or dashboard points at [Exposed components](exposed-components/index.md); a reachable metadata endpoint points at the cloud, via [Cloud metadata from pod](cluster-enumeration/cloud-metadata-from-pod.md).

## Subtopics

- **[Cluster enumeration](cluster-enumeration/index.md)**: mapping the cluster and your identity from a pod.
- **[Exposed components](exposed-components/index.md)**: unauthenticated or reachable control-plane and node components.
- **[RBAC privilege escalation](rbac-privilege-escalation/index.md)**: turning limited permissions into more.
- **[Pod escape to node](pod-escape-to-node/index.md)**: breaking out of a pod to its node.
- **[Lateral movement](lateral-movement/index.md)**: moving between workloads, namespaces, and into the cloud.
- **[Persistence](persistence/index.md)**: keeping access across restarts and remediation.
- **[Network](network/index.md)**: attacking the cluster network and service mesh.
- **[Supply chain](supply-chain/index.md)**: Helm, operators, and admission as deployment paths.

## References

- [Kubernetes security concepts](https://kubernetes.io/docs/concepts/security/)
- [MITRE ATT&CK: Containers matrix](https://attack.mitre.org/matrices/enterprise/containers/)
