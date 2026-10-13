---
title: "Block and SCSI: escaping QEMU through the storage controllers"
order: 4
description: "QEMU's storage path spans the emulated IDE and AHCI controllers and the paravirtual virtio-blk and virtio-scsi devices. The guest programs command structures and DMA descriptors that QEMU reads to perform disk I/O, so flaws in command parsing, in AHCI command-list and PRD handling, and in SCSI request processing give out-of-bounds access in the QEMU process."
keywords:
  - ahci
  - ide
  - virtio-blk
  - virtio-scsi
  - prd
---

# Block and SCSI

A QEMU guest's disk is served by an emulated storage controller: legacy IDE, the AHCI SATA controller, or the paravirtual virtio-blk and virtio-scsi devices. Each takes guest-programmed command structures and DMA descriptors and reads them to perform I/O. AHCI uses a command list and physical region descriptor (PRD) tables in guest memory; SCSI paths parse command descriptor blocks and transfer lengths; virtio-blk/scsi use the virtqueue descriptor model. Parsing those guest-controlled structures in QEMU is the escape surface, with AHCI command-list/PRD handling and SCSI request parsing the recurring loci.

## The surface

```c
// AHCI: the guest sets the command list base; each command header points at a
// command table with a PRD (physical region descriptor) table. QEMU walks the PRDs,
// each giving a guest address and byte count, to DMA data. Primitives:
//  - a PRD count/length QEMU trusts for a transfer -> OOB read/write
//  - a command-list/command-table offset the model uses without bounding
// SCSI (scsi-disk / virtio-scsi): a CDB and transfer length; a length used for a
//  buffer operation beyond the allocation, or an unexpected command, -> OOB
// virtio-blk/scsi: request headers over the virtqueue (see virtio devices)
```

```bash
lspci -nn | grep -iE 'sata|ahci|scsi|virtio'   # which storage controller
```

## Exploitation notes

- AHCI PRD handling is a classic target: the guest fully controls the PRD table (addresses and counts) that QEMU uses for DMA, so a trusted count or length is a direct out-of-bounds transfer primitive.
- SCSI request parsing (shared by scsi-disk and virtio-scsi backends) handles guest-chosen command blocks and transfer lengths, another place length assumptions break.
- virtio-blk/scsi route through the shared virtqueue layer; the storage-specific parsing is the request header, see [virtio devices](virtio-devices.md).
- The guest programs these structures directly from its storage driver; primitives land in QEMU, QEMU-version-specific.

## References

- [QEMU storage documentation](https://www.qemu.org/docs/master/system/devices/)
- [AHCI (SATA) specification](https://www.intel.com/content/www/us/en/io/serial-ata/ahci.html)
- [QEMU security advisories](https://www.qemu.org/docs/master/system/security.html)
