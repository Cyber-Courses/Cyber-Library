---
title: "Synthetic devices: escaping Hyper-V through the VSP-backed devices"
description: "Hyper-V synthetic devices (storage, network, video, HID, and others) are paravirtual devices whose guest-side drivers talk to Virtualization Service Providers in the root partition over VMBus. The VSPs parse guest-supplied requests and descriptors, so a memory-safety flaw in a VSP, notably the synthetic video and storage providers, gives code execution in the privileged root partition."
keywords:
  - synthetic devices
  - vsp
  - storvsp
  - synthvid
  - root partition
---

# Synthetic devices

Hyper-V's performant I/O uses synthetic devices: the guest runs a Virtualization Service Consumer (VSC) driver that communicates over VMBus with a Virtualization Service Provider (VSP) in the root partition. There are synthetic storage, network, video, keyboard/mouse (HID), and other providers. Each VSP parses the requests and descriptors its guest VSC sends, so each is an escape surface, and the synthetic video (`synthvid`) and storage (`storvsp`) providers have historically been productive because of the complexity of what they parse.

## The per-device surface

```c
// a synthetic device request carries a device-specific message over VMBus; the VSP
// in the root partition parses it. Typical primitives:
//  - synthetic video: a message describing a framebuffer/mode with dimensions or a
//    length the VSP trusts, or a situation update referencing a region out of range
//  - synthetic storage (storvsp): a SCSI request packet whose data-transfer length
//    or scatter-gather description the provider uses without bounds checking
//  - synthetic HID/keyboard: report descriptors and input reports parsed in the host
```

The guest fully controls these messages, so the bug classes are the familiar ones: a length or count the VSP trusts, an index it does not bound, or a use-after-free on a request object across the channel lifecycle, all landing in root-partition memory.

## Exploitation notes

- The synthetic video VSP is a repeated target because it parses mode and region descriptors with guest-controlled geometry; the synthetic storage VSP parses SCSI-level requests with guest-controlled transfer descriptions.
- VSPs run in the root partition (several as kernel components), so corruption is high-privilege; which VSP you hit determines whether you land in a kernel VSP or the worker process.
- The guest reaches a given VSP by driving its VSC over VMBus; from an attacker-controlled guest driver you issue crafted device messages directly rather than through the normal stack.
- These are version-specific; match the Hyper-V build to the VSP advisory, and note the transport beneath them, [VMBus](vmbus.md), is a separate surface.

## References

- [Microsoft: Hyper-V synthetic devices and VSP/VSC](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [MSRC: Hyper-V VSP research](https://www.microsoft.com/en-us/msrc)
