---
order: 1
title: "Accès à l'hôte et shell : exécution sur l'hôte bhyve"
description: "Un hôte bhyve est une machine FreeBSD exécutant les processus bhyve. L'accès provient d'une évasion d'invité atterrissant dans un processus bhyve, de SSH avec des identifiants hôte, ou de l'outillage de gestion (bhyve/bhyvectl, ou un wrapper comme vm-bhyve). Avec un accès hôte, un attaquant contrôle chaque VM, lit les images disque des invités et persiste sur l'hôte FreeBSD."
keywords:
  - hôte bhyve
  - freebsd
  - bhyvectl
  - vm-bhyve
  - accès hôte
---

# Accès à l'hôte et shell

Un hôte bhyve est un serveur FreeBSD ordinaire exécutant un processus `bhyve` par VM, souvent orchestré par un wrapper tel que `vm-bhyve` ou intégré à une plateforme (par exemple une appliance de stockage). Atteindre un shell provient d'une évasion guest-to-host qui atterrit dans un processus `bhyve`, de SSH avec des identifiants hôte, ou de l'outillage de gestion. L'accès hôte contrôle chaque VM via `bhyvectl` et le wrapper, expose toutes les images disque des invités et leurs données, et permet la persistance sur l'hôte FreeBSD comme sur n'importe quel serveur Unix.

## Atteindre et utiliser l'hôte

```bash
ssh root@<bhyve-host>
ls /dev/vmm/                               # running VMs (one device per VM)
bhyvectl --vm=<name> --get-stats 2>/dev/null
vm list 2>/dev/null; vm info <name> 2>/dev/null   # vm-bhyve wrapper, if present
# guest disk images (the data to steal)
ls -l /dev/zvol/*/* 2>/dev/null            # zvol-backed disks (common on ZFS)
find / -name '*.img' -path '*vm*' 2>/dev/null
```

## Ce que donne l'accès hôte

```bash
# read a guest disk offline (raw image or zvol)
mdconfig -a -t vnode -f /path/guest.img    # attach a raw image as a memory disk
mount -r /dev/md0p2 /mnt                    # mount read-only to read guest files
# zvol-backed disks are block devices under /dev/zvol; attach similarly
# persist on the FreeBSD host: rc scripts, cron, SSH keys, or a periodic script
```

## Notes d'exploitation

- Le privilège post-évasion est celui de l'utilisateur exécutant `bhyve` ; sur un hôte dédié c'est souvent root, mais vérifiez-le, et prévoyez une élévation de privilèges locale si le processus n'est pas privilégié.
- Les disques des invités sont fréquemment des zvols ZFS sur les hôtes bhyve ; lisez-les hors ligne comme périphériques blocs (`/dev/zvol/...`) ou attachez des images brutes avec `mdconfig`, puis montez en lecture seule, contournant les contrôles de l'invité.
- `bhyvectl` et le wrapper `vm-bhyve` contrôlent le cycle de vie des VM ; utilisez-les pour énumérer et manipuler les invités depuis l'hôte.
- La persistance suit les conventions FreeBSD (`/etc/rc.local`, cron, `/usr/local/etc/rc.d`) ; identifiez la version de FreeBSD pour toute exposition de service hôte.

## Références

- [FreeBSD Handbook: bhyve host](https://docs.freebsd.org/en/books/handbook/virtualization/)
- [vm-bhyve](https://github.com/churchers/vm-bhyve)
