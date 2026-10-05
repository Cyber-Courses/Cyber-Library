---
title: "Host access and shell: execution in the Xen dom0"
description: "The Xen control domain, dom0, is a privileged Linux (or other) domain that runs the toolstack and, usually, the device backends. Reaching a shell in dom0, through a guest escape, the toolstack, or SSH, is host control: it administers every domain, accesses all guest disks, and can inject into or recreate guests through the Xen tools."
keywords:
  - dom0
  - xen toolstack
  - xl
  - xenstore
  - host access
---

# Host access and shell

dom0 is Xen's control domain: a privileged OS (commonly Linux) that boots first, runs the toolstack (`xl`/libxl, or XAPI on XenServer/XCP-ng), hosts the device backends, and administers all other domains. Reaching a shell in dom0 is host control. It comes from a guest-to-host escape that lands in dom0 (via a backend) or in the hypervisor, from toolstack or XAPI access, or from SSH with dom0 credentials. From dom0 an attacker controls every domain through the tools, reads all guest disks, manipulates the XenStore configuration database, and persists on the host.

## Reach and use dom0

```bash
# SSH into dom0, or arrive via a backend/hypervisor escape
xl list                                   # all domains
xl console <domid>                        # attach to a guest console
# XenStore holds domain configuration and PV device wiring
xenstore-ls /local/domain                 # per-domain config visible from dom0
# run against guest disks (see disk theft) and recreate/modify domains
xl create /etc/xen/<guest>.cfg
```

## What dom0 gives

```text
dom0 control includes:
- lifecycle control of every domain (create, destroy, pause, console)
- access to all guest virtual disks through the backends/storage (offline theft)
- the XenStore database wiring devices and config for all domains
- on XenServer/XCP-ng, the XAPI management plane and pool-wide control
Persistence is standard for the dom0 OS (systemd/cron/keys), plus domain config under /etc/xen.
```

## Exploitation notes

- dom0 is the privileged domain, so dom0 root is host-level control of the whole Xen system, equivalent to compromising the hypervisor host.
- The toolstack (`xl`) and XenStore are the control surfaces from dom0; XenStore exposes and configures every domain's device wiring, useful for both enumeration and tampering.
- On XenServer/XCP-ng, dom0 also runs XAPI and participates in a resource pool, so dom0 access extends pool-wide; see [Management plane](management-plane.md).
- Persistence follows the dom0 OS; domain configuration files under `/etc/xen` are an additional, virtualization-specific location.

## References

- [Xen toolstack (xl/libxl)](https://xenbits.xen.org/docs/)
- [XenStore documentation](https://wiki.xenproject.org/wiki/XenStore)
