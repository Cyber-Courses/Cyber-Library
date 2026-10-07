---
title: "Guest-to-host escape: breaking out of Workstation and Fusion to the host"
order: 1
description: "A Workstation or Fusion guest escapes by corrupting the vmware-vmx process that emulates its devices, exactly as on ESXi but landing on the user's desktop host. The same SVGA 3D, USB, NIC, and GuestRPC/VMCI surfaces apply, and code execution in vmware-vmx runs with the privileges of the user or service running the VM."
keywords:
  - vmware-vmx
  - guest-to-host
  - svga
  - guestrpc
  - desktop escape
---

# Guest-to-host escape

On Workstation and Fusion each VM is emulated by a `vmware-vmx` process in the host user's session, so a guest-to-host escape is the same memory-corruption problem as on ESXi, with the payoff landing on the desktop host. The reachable surfaces are identical because the code is shared: the SVGA II device and its 3D command stream, the emulated USB controllers, the virtual NICs, and the backdoor GuestRPC and VMCI control channels. A memory-safety flaw in any of them gives code execution in `vmware-vmx`, which runs with the privileges of whoever started the VM.

```bash
# the escape surface, enumerated from inside the guest (same as ESXi)
lspci -nn | grep -iE 'svga|vmxnet|usb|vmci'
lsusb; dmesg | grep -iE 'vmwgfx|vmxnet|vmw_vmci'
```

The per-device mechanisms are documented under the ESXi guest-to-host pages, since the device models are shared:

- SVGA 3D command-stream parsing, see [SVGA and 3D graphics](../esxi/guest-to-host-escape/svga-and-3d-graphics.md).
- USB controller descriptor and ring handling, see [USB controllers](../esxi/guest-to-host-escape/usb-controllers.md).
- Virtual NIC ring and descriptor handling, see [Virtual NICs](../esxi/guest-to-host-escape/virtual-nics.md).
- The backdoor GuestRPC and VMCI channels, see [Backdoor and VMCI](../esxi/guest-to-host-escape/backdoor-and-vmci.md).

## What differs from ESXi

```bash
# the host is a desktop OS, so post-escape you are in the user's session:
#  - Windows: vmware-vmx.exe runs as the logged-in user (or elevated if the VM was)
#  - macOS/Linux: vmware-vmx runs as the user; Fusion uses a helper for some ops
# privilege after escape = the VM-launching user's, then local privesc on the host
```

## Exploitation notes

- The device-model bug classes are shared with ESXi, so the mechanisms and the enumeration are identical; the difference is the landing environment, a desktop user session rather than the vmkernel host.
- Post-escape privilege equals the user running the VM; a further local privilege escalation on the desktop OS is often needed for full host control, unlike ESXi where `vmx` is already highly privileged.
- The integration features (shared folders, drag-and-drop) are more commonly enabled on desktop products than on servers, making the GuestRPC/HGFS surface especially relevant here; see [Shared folder and drag-drop abuse](shared-folder-and-drag-drop-abuse.md).

## References

- [Zero Day Initiative: VMware Workstation/Fusion escapes](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
