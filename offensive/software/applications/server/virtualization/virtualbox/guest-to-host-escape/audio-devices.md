---
title: "Audio devices: escaping VirtualBox through audio emulation"
description: "Escaping a VirtualBox guest through its audio device emulation (AC97, Intel HDA, SB16), which processes guest-controlled DMA and buffer descriptors in the host-side VM process."
keywords:
  - AC97
  - Intel HDA
  - SB16
  - audio device
  - VirtualBox escape
---

# Audio devices

VirtualBox emulates the AC97, Intel HDA, and SB16 audio devices. They drive DMA engines that move sample data between guest memory and the host audio backend, with the guest controlling the buffer descriptors and DMA parameters. Flaws in that processing corrupt memory in the host-side VM process.

```text
Audio escape surface:
- Intel HDA: command/response rings and stream DMA
- AC97 and SB16: buffer-descriptor and DMA handling
```

## Exploitation notes

- The Intel HDA model is the richest surface and has produced escapes.
- Reachability requires an audio device on the guest, commonly present in desktop and lab VMs.
- Code execution lands in the host VM process, then escalates.

## References

- [Oracle VirtualBox manual](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox research](https://www.zerodayinitiative.com/blog)
