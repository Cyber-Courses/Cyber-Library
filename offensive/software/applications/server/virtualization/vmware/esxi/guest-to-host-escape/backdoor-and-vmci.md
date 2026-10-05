---
title: "Backdoor and VMCI: escaping ESXi through the control channels"
description: "Escaping ESXi through the VMware backdoor RPC channel and the Virtual Machine Communication Interface (VMCI), the control paths between the guest (and VMware Tools) and the vmx process, which parse guest-supplied requests in the host-side process."
keywords:
  - backdoor RPC
  - VMCI
  - vmx
  - RPCI
  - ESXi escape
---

# Backdoor and VMCI

VMware exposes control channels between the guest and the `vmx` process: the legacy backdoor I/O port (used by the RPCI/GuestRPC protocol for Tools features like Shared Folders and drag-and-drop) and the Virtual Machine Communication Interface (VMCI) datagram and socket transport. Both parse guest-supplied requests in the host-side process, so flaws in the request handling yield code execution in `vmx`.

```text
Control-channel escape surface:
- The backdoor port and the GuestRPC / RPCI command handlers
- VMCI datagrams and the vSockets transport
```

## Exploitation notes

- These channels are reachable from the guest without VMware Tools for the raw interface, and with Tools for the richer feature RPCs.
- The GuestRPC handlers back the HGFS and drag-and-drop features, overlapping the desktop products' integration surface.
- Code execution lands in the `vmx` process, then escalates to the host.

## References

- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
