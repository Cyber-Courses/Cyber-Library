---
title: "Xen: attacking the bare-metal paravirtualization hypervisor"
description: "Xen is a type-1 hypervisor where a privileged dom0 hosts the control stack and the backend drivers for guest I/O. The guest-to-host escape targets the hypercall interface, the grant tables and event channels, and the paravirtual backend drivers running in dom0, alongside theft of guest disks and the toolstack management plane."
keywords:
  - xen
  - dom0
  - hypercall
  - grant tables
  - paravirtual
---

# Xen

Xen is a type-1 hypervisor: a thin hypervisor runs on the hardware, and a privileged control domain (dom0) hosts the toolstack and, for most configurations, the backend halves of the guests' paravirtual devices. Guests (domUs) reach host functionality through hypercalls to the hypervisor and through the split paravirtual driver model, where a frontend in the guest talks to a backend in dom0 (or a driver domain) using shared memory via grant tables and notifications via event channels. The escape surface is therefore the hypercall interface, the grant-table and event-channel machinery, and the backend drivers, plus the usual guest-disk theft and the management toolstack.

```bash
# in a Xen guest
dmesg | grep -i xen; ls /sys/bus/xen* 2>/dev/null
cat /sys/hypervisor/type 2>/dev/null       # "xen"
# in dom0: the toolstack and domains
xl list 2>/dev/null
```

## Subtopics

- **[Guest-to-host escape](guest-to-host-escape/index.md)**: hypercalls, grant tables and event channels, and backend drivers.
- **[Host access and shell](host-access-and-shell.md)**: execution in dom0.
- **[Management plane](management-plane.md)**: the Xen toolstack and XAPI.
- **[Disk and snapshot theft](disk-and-snapshot-theft.md)**: taking guest virtual disks.
- **[Known escape exploits](known-escape-exploits.md)**: the recurring hypercall and backend bugs.

## References

- [Xen Project documentation](https://xenbits.xen.org/docs/)
- [Xen security advisories (XSA)](https://xenbits.xen.org/xsa/)
- [Xen architecture](https://wiki.xenproject.org/wiki/Xen_Project_Software_Overview)
