---
title: "Backdoor and VMCI: escaping ESXi through the control channels"
order: 4
description: "The VMware backdoor is an I/O port the guest uses to issue GuestRPC commands to the vmx process, backing Tools features like shared folders and drag-and-drop; VMCI is a paravirtual device for datagrams and vSockets. Both parse guest-controlled requests inside vmx, so flaws in the RPC command handlers or VMCI datagram processing give code execution in the host process."
keywords:
  - backdoor port
  - guestrpc
  - rpci
  - vmci
  - hgfs
---

# Backdoor and VMCI

VMware exposes two control paths from the guest into the host-side `vmx` process. The legacy backdoor is an I/O port, `0x5658`, that the guest drives by loading the magic value `0x564D5868` ("VMXh") into EAX with a command selector in ECX; it carries the GuestRPC/RPCI protocol that implements VMware Tools features. VMCI, the Virtual Machine Communication Interface, is an emulated PCI device providing datagrams and a vSockets transport. Both consume guest-supplied data in `vmx`, so a parsing flaw in an RPC command handler (shared folders, drag-and-drop, copy-paste) or in VMCI datagram handling corrupts `vmx` memory and leads to host code execution.

## The backdoor and GuestRPC

The backdoor is reached with a specific register convention; the guest sets the magic and command and executes an `in`/`out` on the port:

```c
// issue a backdoor command (simplified): magic in EAX, port 0x5658, cmd in CX
static inline void vmware_backdoor(uint32_t cmd, uint32_t arg,
                                   uint32_t *eax, uint32_t *ebx) {
    uint32_t a, b, c=cmd, d=0x5658;
    asm volatile("in %%dx, %%eax"
                 : "=a"(a), "=b"(b)
                 : "a"(0x564D5868), "b"(arg), "c"(c), "d"(d));
    *eax=a; *ebx=b;
}
// GuestRPC is layered on top: open a channel, then send RPCI command strings
// (BDOOR_CMD_MESSAGE), e.g. "info-get guestinfo.x" or the HGFS/DnD feature RPCs
```

The high-value handlers are the feature RPCs: HGFS (shared folders) parses a request protocol in `vmx`, and the drag-and-drop and copy-paste (DnD/CP) handlers have historically carried the memory-safety bugs used in guest-to-host escapes. A guest issues crafted RPCI messages to drive a vulnerable handler.

## VMCI

VMCI is a PCI device the guest driver programs to send datagrams and establish queue pairs and vSocket connections. The host side processes datagram headers and queue-pair setup from guest-controlled memory:

```bash
# the guest VMCI device and driver
lspci -nn | grep -i vmci
dmesg | grep -i vmw_vmci
# datagrams are sent via the device's hypercall/MMIO interface the driver exposes
```

Flaws in VMCI datagram or queue-pair handling, and in the vSockets layer, corrupt `vmx` (or in some paths the kernel module) from the guest.

## Exploitation notes

- The raw backdoor port is reachable without VMware Tools, so the base GuestRPC surface is available to any guest that can execute privileged I/O; the richer feature RPCs assume Tools but a guest can speak the protocol directly.
- HGFS and DnD/CP handlers are the classic targets because they parse complex, nested guest structures in `vmx`; a bug there yields an out-of-bounds or use-after-free in the host process heap.
- A full escape chains an information leak (to defeat ASLR in `vmx`) with the memory-corruption primitive, then pivots from code execution in `vmx` to the host; these are version-specific, so fingerprint the ESXi build first.
- The desktop products expose the same GuestRPC/HGFS surface, which is why shared-folder bugs recur across Workstation, Fusion, and ESXi; see [Workstation and Fusion](../../workstation-and-fusion/index.md).

## References

- [Zero Day Initiative: VMware GuestRPC and vmx research](https://www.zerodayinitiative.com/blog)
- [VMware backdoor protocol (community documentation)](https://sites.google.com/site/chitchatvmback/backdoor)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
