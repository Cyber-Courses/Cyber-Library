---
title: "Seccomp ou AppArmor non confinés : évasions débloquées par un appel système ou un profil LSM désactivé"
description: "Un conteneur lancé avec seccomp=unconfined ou sans profil AppArmor/SELinux perd les filtres qui bloquent normalement les appels système et les chemins de fichiers dangereux. Cela réactive mount, ptrace, keyctl, unshare, et les écritures vers des chemins proc et sysfs qu'un profil par défaut refuse, transformant des évasions par capacité autrement bloquées en évasions fonctionnelles."
keywords:
  - seccomp unconfined
  - apparmor unconfined
  - profil docker-default
  - filtre d'appels système
  - évasion de conteneur
---

# Seccomp ou AppArmor non confinés

Seccomp et les Linux Security Modules (AppArmor ou SELinux) sont la seconde ligne après les capacités : même une capacité que vous détenez est inutile si l'appel système qui la sous-tend est filtré, ou si le chemin dont elle a besoin est refusé. Docker fournit un profil seccomp par défaut qui bloque une quarantaine d'appels système et un profil AppArmor `docker-default` qui refuse les écritures vers des chemins `proc` et `sys` sensibles. Lancer avec `--security-opt seccomp=unconfined` ou `--security-opt apparmor=unconfined` supprime ces filtres, rouvrant des voies d'évasion qu'un conteneur par défaut ne peut pas utiliser.

Identifiez le confinement :

```bash
grep Seccomp /proc/self/status          # 0 = no filter, 2 = a filter is active
grep Seccomp_filters /proc/self/status  # number of installed filters
cat /proc/self/attr/current 2>/dev/null # "docker-default (enforce)" or "unconfined"
cat /sys/kernel/security/apparmor/profiles 2>/dev/null | head
```

## Ce que le mode non confiné réactive

Le profil seccomp par défaut bloque, entre autres, `mount`, `umount2`, `pivot_root`, `ptrace`, `keyctl`, `add_key`, `bpf`, `init_module`/`finit_module`, `kexec_load`, et les drapeaux d'espace de noms `clone`/`unshare` pour les espaces de noms utilisateur. Avec `seccomp=unconfined`, un conteneur qui détient aussi la capacité correspondante peut désormais réellement les appeler :

```bash
# with seccomp=unconfined + CAP_SYS_ADMIN, mount is callable again
mount -t cgroup -o rdma cgroup /tmp/cg 2>&1   # fails under default seccomp, works unconfined
# with seccomp=unconfined + CAP_SYS_MODULE, module loading is callable
insmod ./evil.ko
# ptrace-based host injection becomes possible with a shared PID namespace
strace -p <host_pid>
```

AppArmor `docker-default`, lorsqu'il est appliqué, refuse en outre les écritures vers des chemins tels que `/proc/sysrq-trigger`, `/proc/sys/kernel/*` et `/sys/firmware/*` même si le DAC et les capacités les autoriseraient. Lancer `apparmor=unconfined` lève ces refus de chemin :

```bash
# denied under docker-default, allowed unconfined: trigger a host action via sysrq
echo c > /proc/sysrq-trigger           # (crash) example of a path the profile blocks
echo <hostpid> > /sys/fs/cgroup/.../cgroup.procs   # cgroup writes the profile may deny
```

## Notes d'exploitation

- Les profils non confinés s'évadent rarement à eux seuls ; ils sont un amplificateur. L'évasion a toujours besoin de la capacité ou du périphérique. La combinaison qui compte est, par exemple, `CAP_SYS_ADMIN` plus `seccomp=unconfined`, qui réactive la voie [cgroups release_agent](cgroups-release-agent.md) fondée sur le montage.
- Vérifiez toujours les deux filtres : un conteneur peut conserver des capacités tout en étant neutralisé par seccomp, ou détenir un profil AppArmor appliqué qui bloque précisément l'écriture `proc`/`sys` dont votre technique a besoin. `Seccomp: 2` avec un faible nombre de filtres plus `docker-default (enforce)` est une cible durcie.
- `--privileged` positionne implicitement les deux en mode non confiné, c'est pourquoi il déverrouille toutes les voies ; voir [Flag privileged](privileged-flag.md).

## Références

- [Docker : seccomp security profiles](https://docs.docker.com/engine/security/seccomp/)
- [Docker : AppArmor security profiles](https://docs.docker.com/engine/security/apparmor/)
- [moby default seccomp profile](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)
