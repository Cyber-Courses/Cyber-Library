---
title: "3D acceleration: escaping VirtualBox through the accelerated-graphics path"
description: "VirtualBox 3D acceleration exposes a graphics command channel from the guest (via Guest Additions) to the host VM process, historically the Chromium-based OpenGL passthrough and the VMSVGA device. The host parses a guest-controlled command and shader stream, so flaws in that parsing yield out-of-bounds writes in the VM process, one of VirtualBox's most productive escape surfaces."
keywords:
  - 3d acceleration
  - vmsvga
  - chromium
  - opengl
  - shader
---

# 3D acceleration

When 3D acceleration is enabled, the VirtualBox guest (through Guest Additions) sends a graphics command stream to the host VM process, which interprets it to render with the host GPU. Two implementations have existed: the older Chromium-based OpenGL command passthrough, and the VMSVGA device. Both parse a rich, guest-controlled stream, OpenGL command packets or SVGA/3D commands with surfaces, shaders, and buffers, in the host process, making 3D one of VirtualBox's most productive escape surfaces, with repeated out-of-bounds and type-confusion bugs.

## The surface

```c
// with 3D enabled, the guest submits graphics commands over the Additions channel.
// Chromium path: HGCM-delivered OpenGL command packets with guest-chosen opcodes,
//   array sizes, and buffer lengths the host parser trusts.
// VMSVGA path: an SVGA FIFO/command stream (surfaces, shaders, DMA via GMRs) similar
//   to the VMware SVGA model. Primitives:
//    - a command with a length/count the host uses without bounding -> OOB write
//    - a shader or surface defined with sizes that overflow a host allocation
//    - an object id referenced across commands without validation -> type confusion
```

```bash
# 3D must be enabled on the VM (VBoxManage modifyvm <vm> --accelerate3d on) and the
# Guest Additions 3D driver loaded; confirm in the guest:
lsmod | grep -i vboxvideo; glxinfo 2>/dev/null | head
```

## Exploitation notes

- 3D must be enabled and the Guest Additions graphics driver loaded for the richest surface; this is common on desktop VMs configured for graphics.
- The VMSVGA path mirrors the VMware SVGA 3D model (FIFO commands, surfaces, shaders, GMR DMA), so the bug classes and the [VMware SVGA](../../vmware/esxi/guest-to-host-escape/svga-and-3d-graphics.md) analysis carry over; the Chromium path is an OpenGL command parser with its own array/length bugs.
- Primitives land in the VM process; a full escape pairs a leak with the write and is version-specific.
- The command stream is driven from a controlled guest graphics driver, giving precise control of the parsed structures.

## References

- [VirtualBox manual: 3D acceleration](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox 3D research](https://www.zerodayinitiative.com/blog)
- [Oracle security alerts](https://www.oracle.com/security-alerts/)
