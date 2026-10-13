---
title: "Audio devices: escaping QEMU through emulated sound cards"
order: 6
description: "QEMU emulates sound cards, the Intel HD Audio controller, AC97, Sound Blaster 16, and ES1370, whose guest drivers program buffer descriptors and registers that QEMU reads to move audio data. Flaws in buffer-descriptor handling and DMA length processing, notably in the Intel HDA controller, give out-of-bounds access in the QEMU process."
keywords:
  - intel hda
  - ac97
  - sound blaster
  - audio
  - buffer descriptor
---

# Audio devices

Audio is an easily overlooked escape surface. QEMU emulates several sound cards, the Intel HD Audio controller (`intel-hda`), AC97, Sound Blaster 16, and ES1370, each driven by guest register writes and, for HDA, buffer descriptor lists in guest memory. The HDA controller uses a ring of buffer descriptors (a BDL) describing DMA buffers for stream data, and QEMU walks those descriptors to move audio, so a guest-controlled descriptor length or DMA setup that QEMU trusts is an out-of-bounds primitive. The simpler cards expose register and DMA handling with the same class of length-trust bugs.

## The surface

```c
// Intel HDA: the guest programs stream descriptors and a buffer descriptor list
// (BDL) base; each BDL entry gives a guest address and length for a DMA buffer.
// QEMU walks the BDL to transfer stream data. Primitives:
//  - a BDL entry length/count QEMU uses for a transfer beyond the buffer -> OOB
//  - stream/position registers driving an index the model does not bound
// AC97/SB16/ES1370: DMA buffer registers and lengths with the same trusted-length class
```

```bash
lspci -nn | grep -i audio       # intel-hda / AC97 / ES1370 present
# the guest audio driver programs the controller; an attacker driver sets the BDL
# and stream registers directly with controlled addresses and lengths
```

## Exploitation notes

- The Intel HDA controller is the richest audio target because of its buffer-descriptor-list DMA model; a crafted BDL with trusted lengths drives an out-of-bounds transfer in QEMU.
- Audio devices are frequently present in default desktop-style VM configurations, so the surface exists without special setup; confirm with `lspci`.
- The guest controls the descriptor and register programming from its driver; exploitation sets up the DMA structures directly rather than playing sound normally.
- Primitives land in the QEMU process; QEMU-version-specific, pair with a leak and groom.

## References

- [QEMU audio documentation](https://www.qemu.org/docs/master/system/devices/)
- [Intel High Definition Audio specification](https://www.intel.com/content/www/us/en/standards/high-definition-audio-specification.html)
- [QEMU security advisories](https://www.qemu.org/docs/master/system/security.html)
