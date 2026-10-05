---
title: "Host access and shell: execution on the bhyve host"
description: "A bhyve host is a FreeBSD machine running the bhyve processes. Access comes from a guest escape landing in a bhyve process, from SSH with host credentials, or from the management tooling (bhyve/bhyvectl, or a wrapper like vm-bhyve). With host access an attacker controls every VM, reads the guest disk images, and persists on the FreeBSD host."
keywords:
  - bhyve host
  - freebsd
  - bhyvectl
  - vm-bhyve
  - host access
---

# Host access and shell

A bhyve host is an ordinary FreeBSD server running one `bhyve` process per VM, often orchestrated by a wrapper such as `vm-bhyve` or integrated into a platform (for example a storage appliance). Reaching a shell comes from a guest-to-host escape that lands in a `bhyve` process, from SSH with host credentials, or from the management tooling. Host access controls every VM through `bhyvectl` and the wrapper, exposes all the guest disk images and their data, and allows persistence on the FreeBSD host like any Unix server.

## Reach and use the host

```bash
ssh root@<bhyve-host>
ls /dev/vmm/                               # running VMs (one device per VM)
bhyvectl --vm=<name> --get-stats 2>/dev/null
vm list 2>/dev/null; vm info <name> 2>/dev/null   # vm-bhyve wrapper, if present
# guest disk images (the data to steal)
ls -l /dev/zvol/*/* 2>/dev/null            # zvol-backed disks (common on ZFS)
find / -name '*.img' -path '*vm*' 2>/dev/null
```

## What host access gives

```bash
# read a guest disk offline (raw image or zvol)
mdconfig -a -t vnode -f /path/guest.img    # attach a raw image as a memory disk
mount -r /dev/md0p2 /mnt                    # mount read-only to read guest files
# zvol-backed disks are block devices under /dev/zvol; attach similarly
# persist on the FreeBSD host: rc scripts, cron, SSH keys, or a periodic script
```

## Exploitation notes

- Post-escape privilege is that of the user running `bhyve`; on a dedicated host that is often root, but confirm, and plan a local privilege escalation if the process is unprivileged.
- Guest disks are frequently ZFS zvols on bhyve hosts; read them offline as block devices (`/dev/zvol/...`) or attach raw images with `mdconfig`, then mount read-only, bypassing guest controls.
- `bhyvectl` and the `vm-bhyve` wrapper control the VM lifecycle; use them to enumerate and manipulate guests from the host.
- Persistence follows FreeBSD conventions (`/etc/rc.local`, cron, `/usr/local/etc/rc.d`); fingerprint the FreeBSD version for any host-service exposure.

## References

- [FreeBSD Handbook: bhyve host](https://docs.freebsd.org/en/books/handbook/virtualization/)
- [vm-bhyve](https://github.com/churchers/vm-bhyve)
