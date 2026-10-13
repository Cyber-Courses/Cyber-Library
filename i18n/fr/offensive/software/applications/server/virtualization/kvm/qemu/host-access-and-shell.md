---
order: 1
title: "Accès à l'hôte et shell : exécution sur l'hôte QEMU/KVM"
description: "Un hôte KVM est une machine Linux exécutant des processus QEMU sous libvirt. L'accès à l'hôte provient d'une évasion depuis l'invité atterrissant dans un processus QEMU, d'un accès libvirt ou SSH, ou d'identifiants hôte. Parce que QEMU peut s'exécuter confiné par seccomp et sous un utilisateur dédié, l'étape post-évasion est souvent une élévation de privilèges locale vers le contrôle total de l'hôte sur chaque VM."
keywords:
  - kvm host
  - qemu process
  - libvirt
  - seccomp
  - privilege escalation
---

# Accès à l'hôte et shell

Un hôte KVM est un serveur Linux ordinaire exécutant un processus QEMU par VM, généralement géré par libvirt. Atteindre l'hôte se fait de trois façons : une évasion guest-to-host qui place du code dans un processus QEMU, un accès libvirt ou SSH avec des identifiants hôte, ou une compromission d'un service côté hôte. Ce qu'une évasion QEMU donne dépend de la façon dont le processus est confiné : libvirt exécute couramment QEMU sous un utilisateur non privilégié dédié (`qemu`/`libvirt-qemu`), sous un sandbox seccomp, et parfois avec SELinux/AppArmor (sVirt), si bien qu'atterrir dans QEMU est fréquemment un point d'appui contraint qui nécessite tout de même une élévation de privilèges locale pour atteindre le contrôle total de l'hôte.

## D'un point d'appui QEMU vers l'hôte

```bash
# determine the confinement of the QEMU process you landed in
id                                   # dedicated qemu/libvirt-qemu user?
grep Seccomp /proc/self/status       # seccomp filter active?
cat /proc/self/attr/current 2>/dev/null   # SELinux/AppArmor (sVirt) label
# enumerate what the qemu user can reach: other VMs' images, the libvirt socket
ls -l /var/lib/libvirt/images/ /var/run/libvirt/libvirt-sock 2>/dev/null
```

```bash
# with libvirt/SSH host access instead of an escape:
virsh list --all; virsh dumpxml <dom>          # control every VM
ssh root@<kvm-host>                            # full host if creds allow
```

## Ce que donne le contrôle de l'hôte

```bash
# every VM's disk is readable (see disk-and-snapshot theft), and the libvirt API
# controls all domains; persist on the host like any Linux server:
#  - a systemd unit, cron, or SSH key
#  - a libvirt hook (/etc/libvirt/hooks/qemu) runs on VM lifecycle events as root
```

## Notes d'exploitation

- Le confinement post-évasion est la variable clé : un processus QEMU s'exécutant sous un utilisateur dédié avec seccomp/sVirt signifie que l'évasion donne ce contexte limité, alors prévoyez une élévation de privilèges locale via le noyau ou un service hôte pour atteindre root.
- L'accès hôte libvirt ou SSH est la voie la plus simple lorsqu'elle est disponible et englobe une évasion depuis l'invité : il contrôle chaque domaine et lit chaque image directement.
- Les hooks libvirt (`/etc/libvirt/hooks/qemu`) s'exécutent en root lors des événements de cycle de vie des VM et constituent un emplacement de persistance hôte propre, spécifique à KVM, à côté des mécanismes Linux standard.
- Le contrôle de l'hôte englobe le [Vol de disques et de snapshots](disk-and-snapshot-theft.md) et le [Plan de gestion](management-plane.md) pour chaque VM.

## Références

- [Sécurité libvirt (sVirt) et sandboxing](https://libvirt.org/drvqemu.html#security)
- [Sécurité QEMU et seccomp](https://www.qemu.org/docs/master/system/security.html)
- [Hooks libvirt](https://libvirt.org/hooks.html)
