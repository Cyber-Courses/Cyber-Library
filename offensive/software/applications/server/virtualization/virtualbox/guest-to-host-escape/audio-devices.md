---
title: "Audio devices: escaping VirtualBox through emulated sound controllers"
description: "VirtualBox emulates the AC97, SB16, and Intel HD Audio controllers, whose guest drivers program buffer descriptors and DMA that the host VM process reads to move audio. Flaws in buffer-descriptor and DMA length handling, particularly in the Intel HDA controller, give out-of-bounds access in the host process from the guest audio stack."
keywords:
  - intel hda
  - ac97
  - sb16
  - audio
  - buffer descriptor
---

# Audio devices

VirtualBox offers several emulated sound cards, the Intel HD Audio controller (ICH6/HDA), AC97, and SoundBlaster 16. As with other hypervisors, audio is an easily overlooked but real escape surface: the HDA controller uses a buffer descriptor list (BDL) in guest memory describing DMA buffers for stream data, and the host walks those descriptors to transfer audio. A guest-controlled descriptor length or DMA setup that the host trusts is an out-of-bounds primitive, and the simpler cards expose register and DMA handling with the same length-trust class.

## The surface

```c
// Intel HDA: the guest programs stream descriptors and a BDL base; each BDL entry
// gives a guest address and length. The host walks the BDL to transfer stream data.
//  - a BDL entry length/count the host uses beyond the buffer -> OOB read/write
//  - stream position/registers driving an index the host does not bound
// AC97/SB16: DMA buffer registers and lengths with the same trusted-length class
```

```bash
lspci -nn | grep -i audio       # HDA / AC97 present
# an attacker guest audio driver programs the BDL and stream registers directly
```

## Exploitation notes

- The Intel HDA controller is the richest audio target because of its BDL-driven DMA model; a crafted BDL with trusted lengths drives an out-of-bounds transfer in the host VM process.
- Audio is frequently present in default desktop VM configurations, so the surface exists without special setup; confirm the controller with `lspci`.
- The guest controls the descriptor and register programming from its driver; exploitation sets up the DMA structures directly rather than playing audio.
- Primitives land in the VM process; version-specific, pair with a leak. The HDA model resembles QEMU's and VMware's, so the bug class recurs across hypervisors.

## References

- [VirtualBox manual: audio](https://www.virtualbox.org/manual/)
- [Intel HD Audio specification](https://www.intel.com/content/www/us/en/standards/high-definition-audio-specification.html)
- [Oracle security alerts](https://www.oracle.com/security-alerts/)
