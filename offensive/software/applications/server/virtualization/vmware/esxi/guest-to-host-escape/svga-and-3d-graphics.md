---
title: "SVGA and 3D graphics: the classic ESXi escape surface"
description: "Escaping ESXi through the SVGA II adapter and its 3D graphics command processing in the vmx process, the most productive VMware escape surface historically, reachable when 3D acceleration is enabled on the guest."
keywords:
  - SVGA
  - 3D graphics
  - vmx
  - shader
  - ESXi escape
---

# SVGA and 3D graphics

The VMware SVGA II adapter and its 3D graphics path are the classic escape surface. When 3D acceleration is enabled, the guest submits graphics commands, shader programs, and surface definitions that the `vmx` process parses and translates. Flaws in that command and object handling corrupt memory in the host-side `vmx` process, and this path has produced the majority of public VMware escapes.

```text
SVGA / 3D escape surface:
- The SVGA command FIFO and register interface
- 3D command processing: shaders, surfaces, contexts
```

## Exploitation notes

- This is the dominant Pwn2Own VMware target; disabling 3D acceleration removes it, so reachability depends on the guest's video configuration.
- The 3D command and object model is large and stateful, which is why it yields so many bugs.
- Code execution lands in the `vmx` process, then escalates to the host; the same code affects Workstation and Fusion.

## References

- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
