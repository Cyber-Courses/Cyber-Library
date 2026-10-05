---
title: "Guest-to-host escape: breaking out of a Parallels virtual machine"
description: "A Parallels guest escapes by corrupting the host-side Parallels processes that emulate the VM's devices and service the guest-host channels. The reachable surface is the emulated devices, graphics, network, USB, and storage, and the Parallels Tools communication interfaces, with a flaw yielding code execution on the macOS host at the privilege of the Parallels process."
keywords:
  - parallels escape
  - device emulation
  - prl
  - macos
  - guest-to-host
---

# Guest-to-host escape

A Parallels VM's devices and integration services are emulated in host-side `prl_*` processes on macOS, so a guest-to-host escape makes one of those processes mishandle guest-controlled data. The surface has two parts: the emulated device models (graphics including accelerated 3D, the network adapter, USB controllers, and storage), which parse guest register writes and DMA structures, and the Parallels Tools communication interfaces, which parse the guest's integration requests. Code execution lands in a Parallels host process on macOS, at that process's privilege.

```bash
# the emulated device inventory in the guest (escape surface)
lspci -nn 2>/dev/null; lsusb 2>/dev/null
dmesg 2>/dev/null | grep -i prl
```

## The surface

```text
Guest-to-host escape surface:
- graphics / 3D acceleration: a guest-controlled command/surface stream parsed host-side
  (the richest device surface, as on other hypervisors' 3D paths)
- network adapter: TX/RX descriptor and offload handling
- USB controllers: transfer-descriptor/ring handling
- storage controller: command and DMA descriptor handling
- Parallels Tools channels: integration requests (see shared-folder/clipboard abuse)
```

The device-model bug classes are the standard ones (trusted lengths, unbounded indices, use-after-free on request objects), and the 3D/graphics path is typically the most productive, as on other hypervisors. The Tools channel parsing is the Parallels-specific integration surface.

## Exploitation notes

- Code execution lands in a Parallels host process on macOS; its privilege determines the immediate impact, with a possible further local privilege escalation on macOS for full host control.
- The graphics/3D path is the recurring device hot spot; the Parallels Tools channels are the integration hot spot, analogous to VMware's GuestRPC and VirtualBox's HGCM.
- The guest drives the device models and Tools channels from controlled drivers/clients, posting crafted structures directly.
- Parallels escapes are version-specific; fingerprint the Parallels Desktop version and match to the advisory, and see the [shared-folder and clipboard](shared-folder-and-clipboard-abuse.md) integration surface.

## References

- [Parallels Desktop documentation](https://www.parallels.com/products/desktop/)
- [Zero Day Initiative: Parallels Pwn2Own escapes](https://www.zerodayinitiative.com/blog)
