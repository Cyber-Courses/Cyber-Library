---
title: "SVGA and 3D graphics: escaping ESXi through the graphics device"
description: "The VMware SVGA II device exposes a command FIFO and a 3D rendering interface that the guest driver fills with commands and surface definitions. The vmx process parses that guest-controlled stream, including shader and surface operations, so a memory-safety flaw in 3D command handling yields an out-of-bounds write in the host process, a recurring guest-to-host escape."
keywords:
  - svga
  - vmwgfx
  - fifo
  - 3d acceleration
  - gmr
---

# SVGA and 3D graphics

The VMware SVGA II device is the guest's display adapter, driven in Linux by `vmwgfx`. It is far more than a framebuffer: the guest submits a stream of commands through a FIFO and, with 3D acceleration enabled, a rich set of rendering operations (surfaces, contexts, shaders, draw calls) that the `vmx` process interprets. Because that command stream and its referenced objects are entirely guest-controlled and parsed in the host process, the 3D path is one of the most productive ESXi escape surfaces, with repeated out-of-bounds and type-confusion bugs in surface and shader handling.

## The command FIFO

The guest enables the FIFO through the device registers and writes command words into the FIFO memory region; `vmx` consumes them:

```c
// SVGA registers are accessed via the index/value I/O ports (SVGA_INDEX/_VALUE)
// enable the device and FIFO, then submit commands into the FIFO memory
svga_write(SVGA_REG_ENABLE, 1);
svga_write(SVGA_REG_CONFIG_DONE, 1);         // FIFO active
// each command begins with an SVGA_CMD_* or SVGA_3D_CMD_* id followed by a body
// 2D example: SVGA_CMD_UPDATE; 3D example: SVGA_3D_CMD_SURFACE_DEFINE, _SHADER_DEFINE,
// _DRAW_PRIMITIVES, each with guest-chosen sizes and object references
```

## The 3D object model

3D rendering defines surfaces (textures and render targets), contexts, and shaders, and issues draw calls referencing them. The host side allocates and tracks these objects from guest-supplied descriptors, so the bug classes are: a surface or mip-level defined with dimensions that overflow an allocation, a shader with a length the parser trusts, a draw call referencing an object id the host dereferences without validation (type confusion), and guest memory regions (GMRs) used for DMA with out-of-range offsets.

```c
// a crafted SURFACE_DEFINE with inconsistent face/mip sizes, or a SHADER_DEFINE
// with an oversized bytecode length, drives the host-side allocation/parse out of
// bounds; DRAW/PRESENT referencing a stale or wrong-typed object id triggers UAF
```

## Exploitation notes

- 3D must be reachable for the richest surface; the device and FIFO are enabled through the SVGA registers from the guest, and the `vmwgfx` driver demonstrates the exact register and FIFO setup sequence.
- The common primitives are an out-of-bounds write from a surface/shader size the host trusts, and a type confusion from an object id referenced across command types; both land in the `vmx` heap.
- GMR-based DMA lets the guest point the device at guest memory with attacker offsets, which combines with the parsing bugs to build a controlled read/write.
- A working escape pairs a leak (surface readback or an info-leak command) to defeat `vmx` ASLR with the write primitive; these are build-specific, so match the ESXi version to a known 3D bug.

## References

- [VMware SVGA device interface (open-vm-tools / vmwgfx headers)](https://github.com/vmware/open-vm-tools)
- [Zero Day Initiative: VMware SVGA 3D research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
