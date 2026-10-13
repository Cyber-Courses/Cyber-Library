---
title: "Modèle rootless : ce qu'un runtime à user namespace change pour l'attaquant"
description: "Podman rootless exécute des conteneurs dans un user namespace où le root du conteneur correspond à l'utilisateur non privilégié appelant sur l'hôte. Une évasion de conteneur rapporte donc les privilèges de cet utilisateur, et non root sur l'hôte, et les capabilities ne s'appliquent qu'au sein du namespace. Comprendre le mapping d'uid détermine quelles évasions valent la peine d'être tentées et ce qu'elles obtiennent."
keywords:
  - podman rootless
  - user namespace
  - uid_map
  - subuid subgid
  - évasion de conteneur
---

# Modèle rootless

Podman rootless s'exécute entièrement en tant qu'utilisateur non privilégié en plaçant les conteneurs dans un user namespace. À l'intérieur, le conteneur voit l'UID 0, mais celui-ci est mappé à travers `/etc/subuid` et `/etc/subgid` vers une plage d'UID d'hôte non privilégiés. La conséquence pour un attaquant est décisive : une évasion de conteneur aboutit à l'utilisateur appelant, et non à root sur l'hôte, et les capabilities qu'un processus de conteneur détient ne sont valides que contre les ressources possédées par la plage mappée. Beaucoup des évasions par capability et par device qui donnent root sur l'hôte sous Docker rootful ne donnent ici qu'une identité confinée et remappée.

Lisez le mapping pour savoir ce qu'une évasion rapporte :

```bash
cat /proc/self/uid_map
#   0     100000     65536   => container root maps to host UID 100000..165535
id; podman info | grep -i rootless
grep "^$USER:" /etc/subuid /etc/subgid                # the allocated ranges
```

## Ce qui fonctionne encore, et ce que cela donne

```bash
# Escaping the container (e.g. via a bad mount) gives the invoking user's access,
# so target that user's secrets, SSH keys, and reachable services, not /etc/shadow.
ls -la $HOME/.ssh $HOME/.config 2>/dev/null
# Capability routes act only within the user namespace: CAP_SYS_ADMIN here cannot
# write host-root-owned files, because the kernel checks ownership against the ns.
# A genuine host-root escape needs a user-namespace-aware kernel bug.
```

## Notes d'exploitation

- Lisez toujours `uid_map` en premier ; une plage mappée (et non `0 0`) signifie que les évasions par capability et par device rapportent l'utilisateur mappé, alors priorisez les propres ressources de l'utilisateur appelant plutôt que les cibles root de l'hôte.
- Le mode rootless élargit tout de même la surface d'attaque du noyau d'une manière : la création du user namespace accorde des capabilities à l'intérieur de celui-ci, qui peuvent atteindre des sous-systèmes (netfilter, certains systèmes de fichiers) pour un exploit noyau qui, lui, escalade vers root sur l'hôte ; voir [nf_tables](../../container-escape/runtime-and-kernel-exploits/kernel-exploits/nf-tables.md).
- Si l'utilisateur sous lequel Podman s'exécute est lui-même privilégié ailleurs (membre de groupes sensibles, détenteur d'identifiants cloud), s'évader vers cet utilisateur reste une position forte même sans root sur l'hôte.

## Références

- [Podman: rootless tutorial](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)
- [Rootless containers](https://rootlesscontaine.rs/)
- [man 7 user_namespaces](https://man7.org/linux/man-pages/man7/user_namespaces.7.html)
