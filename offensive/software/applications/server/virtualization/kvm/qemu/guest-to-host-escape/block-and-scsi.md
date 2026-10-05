---
title: "Block and SCSI: escaping through QEMU storage controllers"
description: "Escaping a KVM guest through the QEMU storage controllers, the AHCI and IDE models and the emulated SCSI adapters (LSI 53c895a, megasas), which parse guest-issued commands and descriptors in the host QEMU process."
keywords:
  - AHCI
  - SCSI
  - LSI megasas
  - storage controller
  - QEMU escape
---

# Block and SCSI

QEMU's storage controllers turn guest disk operations into host I/O, parsing command structures and descriptors from the guest. The AHCI and IDE controllers and the emulated SCSI adapters (the LSI 53c895a and megasas HBAs) have all had memory-corruption flaws in their command and descriptor handling, reachable from a guest that issues crafted storage commands.

```text
Storage escape surface:
- AHCI / IDE command and PRDT handling
- Emulated SCSI HBAs: LSI 53c895a, megasas (command frames)
```

## Exploitation notes

- The emulated SCSI HBAs have a rich command surface and a history of escapes; the controller model on the guest determines which is reachable.
- Command descriptors and scatter-gather lists are the parsed, attacker-controlled structures.
- virtio-block and virtio-scsi are the paravirtualized alternatives; see [virtio devices](virtio-devices.md).

## References

- [QEMU system emulation](https://www.qemu.org/docs/master/system/index.html)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
