---
title: "Floppy controller: escaping QEMU through the legacy FDC"
description: "QEMU's emulated floppy disk controller processes guest commands through a fixed-size internal FIFO. A guest that issues certain read/write commands can drive the FIFO index past the buffer, the VENOM class of bug, giving an out-of-bounds read and write in the QEMU process. The controller is present even when no floppy drive is attached."
keywords:
  - floppy controller
  - fdc
  - venom
  - fifo
  - qemu escape
---

# Floppy controller

QEMU's floppy disk controller (FDC) is a small, old device model that nonetheless produced one of the best-known escapes, VENOM. The controller processes guest commands through a fixed-size internal data register FIFO. For certain commands the handler advances an index into that FIFO as it consumes parameters or transfers data, and a flaw in bounding that index lets a guest push it past the end of the array, reading and writing adjacent QEMU process memory. The controller is instantiated for common machine types even when no floppy image is attached, so the surface is present by default.

## Driving the FDC

```c
// the FDC is programmed through I/O ports around 0x3f0-0x3f5; the guest writes a
// command byte followed by parameter bytes to the data register (0x3f5), then
// reads/writes data for read/write-style commands.
outb(0x3f5, FD_CMD);          // e.g. a read/write or format-track style command
outb(0x3f5, param);           // parameters advance the internal FIFO index
// the vulnerable pattern: a command whose data phase keeps writing into the fixed
// FIFO while the index is not reset/bounded -> write past the FIFO array (VENOM)
```

The bug is that the controller continues to store into the fixed-size FIFO buffer while advancing the index beyond the array bounds for specific commands, so the guest achieves an out-of-bounds write into the surrounding QEMU heap/struct memory, and a corresponding read primitive.

## Exploitation notes

- The controller is present for default PC machine types regardless of whether a floppy drive or image is configured, so this surface does not require any unusual VM configuration.
- The primitive is an out-of-bounds read and write adjacent to the FDC state in the QEMU process; exploitation grooms the heap so useful pointers sit next to the FIFO, then corrupts them for control flow.
- As a legacy single-buffer device it is simple to drive from the guest with raw port I/O, needing only the ability to execute privileged I/O instructions.
- Patched QEMU bounds the FIFO index; this applies to unpatched builds, so fingerprint the QEMU version.

## References

- [QEMU security advisories](https://www.qemu.org/docs/master/system/security.html)
- [VENOM (CrowdStrike advisory)](https://www.crowdstrike.com/blog/venom-vulnerability-details/)
- [Awesome VM escape: FDC](https://github.com/WinMin/Awesome-VM-Exploit)
