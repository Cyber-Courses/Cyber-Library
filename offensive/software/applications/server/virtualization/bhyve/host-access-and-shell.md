---
title: "Host access and shell: reaching the bhyve host"
description: "Reaching the FreeBSD host that runs bhyve: a shell on the host controls every guest through bhyvectl and the bhyve processes, and the guest disk images and ZFS volumes on the host hold guest data at rest."
keywords:
  - bhyve host
  - bhyvectl
  - FreeBSD
  - ZFS volume
  - host access
---

# Host access and shell

A bhyve host is a FreeBSD machine running one `bhyve` process per guest. Control of the host is control of the guests: `bhyvectl` and the management wrappers (vm-bhyve, or an appliance UI) start, stop, and inspect them, and the guest disks are image files or ZFS volumes on the host. Reaching the host is ordinary FreeBSD compromise.

```sh
ls /dev/vmm/                               # running bhyve VMs
bhyvectl --vm=<name> --get-all             # VM state
vm list                                    # vm-bhyve wrapper, if used
zfs list -t volume                         # ZFS-backed guest disks
```

## Exploitation notes

- The host owns every guest's disk image or ZFS volume, so host access leads directly to offline guest data.
- Appliances built on bhyve often expose a management UI or API that fronts the host; compromising it reaches the host shell.
- ZFS-backed guests can be snapshotted and read from the host without touching the running guest.

## References

- [FreeBSD Handbook: bhyve](https://docs.freebsd.org/en/books/handbook/virtualization/#virtualization-host-bhyve)
- [vm-bhyve](https://github.com/churchers/vm-bhyve)
