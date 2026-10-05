---
title: "Host access and shell: reaching the KVM host and libvirt"
description: "Reaching a KVM hypervisor host: a Linux shell on the host and the libvirt control socket, through which virsh manages every VM, their disks, and their consoles. Access to the host or the libvirt socket is control of all guests."
keywords:
  - KVM host
  - libvirt socket
  - virsh
  - qemu
  - host access
---

# Host access and shell

A KVM host is a Linux machine running `qemu-kvm` processes, managed through libvirt (`libvirtd`) and its socket. Control of the host, or of the libvirt socket, is control of every VM: `virsh` lists, starts, stops, and consoles them, and the disk images are on the host filesystem. Membership in the `libvirt` group or access to the socket is enough.

```bash
virsh list --all                          # every guest
virsh dumpxml <vm>                         # config, disk paths, devices
virsh console <vm>                         # guest console
ls /var/lib/libvirt/images/                # qcow2/raw disks
# The socket is the control boundary
ls -l /var/run/libvirt/libvirt-sock
```

## Exploitation notes

- Access to `/var/run/libvirt/libvirt-sock` (often via the `libvirt` group) is full VM control without root on the host.
- `virsh` gives console access and lets you edit a VM to attach a disk or change boot, and the images are directly readable for [Disk and snapshot theft](disk-and-snapshot-theft.md).
- Remote libvirt (`qemu+tcp`/`qemu+ssh`) with weak auth is a network-reachable control path to the same power.

## References

- [libvirt: connection URIs](https://libvirt.org/uri.html)
- [virsh manual](https://www.libvirt.org/manpages/virsh.html)
