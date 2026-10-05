---
title: "Floppy controller: the VENOM guest-to-host escape"
description: "Escaping a KVM guest through the QEMU floppy disk controller, the VENOM class: a command FIFO buffer overflow in the controller's command handling that is reachable even when no floppy drive is attached to the guest, giving code execution in the host QEMU process."
keywords:
  - VENOM
  - floppy disk controller
  - FDC
  - buffer overflow
  - QEMU escape
---

# Floppy controller

The VENOM class exploits QEMU's legacy floppy disk controller (FDC). The controller keeps a fixed command FIFO, and certain commands push more data than the buffer holds, overflowing it with guest-controlled bytes in the host QEMU process. The notorious property is that the FDC command handling is reachable even when the guest has no floppy drive configured, because the controller is still present.

```text
Floppy escape surface:
- The FDC command FIFO and command-parameter handling
- Reachable via FDC I/O ports regardless of an attached floppy image
```

## Exploitation notes

- The defining trait is near-universal reachability on affected versions: the controller responds even without a floppy drive, so hardening the guest does not remove it.
- The overflow is in the host QEMU process; exploitation turns the FIFO overwrite into control of execution.
- This is the canonical example of why an emulated legacy device matters even when unused.

## References

- [CrowdStrike: VENOM technical analysis](https://www.crowdstrike.com/blog/venom-vulnerability-details/)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
