---
order: 4
title: "Évasion de pod vers le nœud : sortir d'un pod sur son worker"
description: "Un pod trop privilégié s'évade vers son nœud worker sans exploit : un pod privilégié, un montage hostPath du système de fichiers du nœud, des namespaces d'hôte partagés ou l'accès aux identifiants du kubelet donnent chacun le root sur le nœud. Posséder le nœud expose ensuite les secrets de tous les autres pods et les identifiants du kubelet qui permettent de pivoter vers le cluster."
keywords:
  - évasion de pod
  - nœud worker
  - pod privilégié
  - hostpath
  - kubelet
---

# Évasion de pod vers le nœud

Un pod est un ensemble de conteneurs sur un nœud worker, et les mêmes mauvaises configurations qui rendent un conteneur évadable rendent possible l'évasion d'un pod vers son nœud. Les voies sont l'expression Kubernetes des primitives d'évasion de conteneur : un pod avec `privileged: true`, un volume `hostPath` montant le système de fichiers du nœud, des namespaces d'hôte partagés, ou des identifiants kubelet accessibles. S'évader vers le nœud a une forte valeur car le nœud exécute chaque pod qui lui est planifié, de sorte que le root sur le nœud expose tous leurs secrets et tokens, et l'identité kubelet propre au nœud permet de pivoter vers le cluster.

Vérifiez le security context du pod pour les voies faciles :

```bash
grep -E 'CapEff|Seccomp' /proc/self/status; cat /proc/self/uid_map
mount | grep -vE 'overlay|proc|sysfs|tmpfs|cgroup' | head    # hostPath mounts
ls /dev | grep -E 'sd|nvme' ; ls /var/run/secrets/kubernetes.io/serviceaccount/
readlink /proc/1/exe                                         # host init => shared PID ns
```

## Sous-sujets

- **[Pod privilégié](privileged-pod.md)** : un pod avec le security context privilégié.
- **[Montage hostPath](hostpath-mount.md)** : un volume montant le système de fichiers du nœud.
- **[Namespaces de l'hôte](host-namespaces.md)** : les pods hostPID, hostNetwork et hostIPC.
- **[Vol d'identifiants du kubelet](kubelet-credential-theft.md)** : voler l'identité kubelet du nœud.

## Références

- [Kubernetes: pod security standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [BishopFox: bad pods](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [Container escape (runtime-agnostic)](../../../container-escape/index.md)
