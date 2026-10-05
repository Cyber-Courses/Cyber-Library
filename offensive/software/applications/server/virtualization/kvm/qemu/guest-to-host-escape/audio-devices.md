---
title: "Audio devices: escaping through QEMU audio emulation"
description: "Escaping a KVM guest through the QEMU audio device models (AC97, Intel HDA, ES1370), which process guest-controlled DMA engines and buffer descriptors in the host QEMU process."
keywords:
  - AC97
  - Intel HDA
  - ES1370
  - audio device
  - QEMU escape
---

# Audio devices

QEMU's audio devices (AC97, Intel High Definition Audio, ES1370) drive DMA engines that move sample data between guest memory and the host audio backend. The guest controls the buffer descriptors and DMA parameters, so flaws in the descriptor processing or stream handling corrupt memory in the host QEMU process.

```text
Audio escape surface:
- Intel HDA: command/response rings and stream DMA descriptors
- AC97 and ES1370: buffer-descriptor and DMA handling
```

## Exploitation notes

- The Intel HDA model has the richest surface (command rings plus stream DMA), and has produced escapes.
- Reachability requires an audio device on the guest, which analysis and desktop VMs commonly have.
- As with other device models, code execution lands in the host QEMU process.

## References

- [QEMU system emulation](https://www.qemu.org/docs/master/system/index.html)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
