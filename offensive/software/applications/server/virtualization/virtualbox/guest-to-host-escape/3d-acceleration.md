---
title: "3D acceleration: escaping VirtualBox through the graphics path"
description: "Escaping a VirtualBox guest through its 3D acceleration path, the Chromium-based and later VMSVGA graphics, which processes guest-submitted graphics commands in the host-side process when 3D acceleration and Guest Additions are enabled."
keywords:
  - 3D acceleration
  - Chromium
  - VMSVGA
  - VirtualBox escape
  - graphics
---

# 3D acceleration

VirtualBox's 3D acceleration forwards guest graphics commands to the host. The legacy path used a Chromium-based protocol, and newer versions use the VMSVGA adapter. Either way the host-side code parses guest-submitted commands and buffers, and flaws there corrupt host memory. The feature requires Guest Additions and 3D acceleration to be enabled.

```text
3D escape surface:
- The Chromium guest-to-host graphics protocol (legacy)
- VMSVGA command and surface handling
```

## Exploitation notes

- Reachability requires 3D acceleration and Guest Additions, which are off by default but common in lab and desktop VMs.
- The graphics command protocol is large and has produced multiple escapes.
- Code execution lands in the host VM process, then escalates.

## References

- [Oracle VirtualBox manual](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox research](https://www.zerodayinitiative.com/blog)
