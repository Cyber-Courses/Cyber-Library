---
title: "Host access and shell: reaching the Xen control domain"
description: "Reaching the Xen control domain (dom0), the privileged Linux domain that manages the hypervisor and every guest, through its shell and the xl or xe toolstack, from where guest consoles, configuration, and virtual disks are fully exposed."
keywords:
  - Xen dom0
  - xl toolstack
  - xe
  - control domain
  - host access
---

# Host access and shell

Xen's dom0 is a privileged Linux domain that drives the hypervisor and owns the guests. Control of dom0 is control of everything: the `xl` toolstack (or `xe` on XCP-ng/XenServer) lists, consoles, and reconfigures domains, and the guest disks live on dom0's storage. Reaching dom0 is ordinary Linux compromise or a management-plane pivot.

```bash
xl list                                   # all domains
xl console <domU>                          # guest console
xl vcpu-list; xl info                      # hypervisor and host info
# XCP-ng / XenServer toolstack
xe vm-list; xe vm-disk-list vm=<name>
```

## Exploitation notes

- dom0 compromise is total: it can start, stop, console, and reconfigure every guest and read their disks for [Disk and snapshot theft](disk-and-snapshot-theft.md).
- On XCP-ng and XenServer, the `xe` toolstack and the xapi service are the control path, often reachable from the [Management plane](management-plane.md).
- dom0 also holds the storage repository metadata, so it maps where every guest disk lives.

## References

- [Xen: the xl command](https://xenbits.xen.org/docs/unstable/man/xl.1.html)
- [XCP-ng documentation](https://docs.xcp-ng.org/)
