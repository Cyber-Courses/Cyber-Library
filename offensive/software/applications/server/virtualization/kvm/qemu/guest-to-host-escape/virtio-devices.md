---
title: "virtio devices: escaping through the paravirtualized device models"
description: "Escaping a KVM guest through the QEMU virtio device models (net, block, scsi, gpu, balloon), which exchange buffers with the guest over the virtqueue shared ring, the broadest modern escape surface since every paravirtualized guest uses them."
keywords:
  - virtio
  - virtqueue
  - virtio-net
  - virtio-gpu
  - QEMU escape
---

# virtio devices

virtio is the paravirtualized device framework every modern guest uses: the guest and QEMU exchange buffers over a shared-memory ring, the virtqueue. The device backends (virtio-net, virtio-block, virtio-scsi, virtio-gpu, virtio-balloon) parse the descriptors and buffers the guest places in that ring, so flaws in descriptor handling or device-specific logic run in the host QEMU process.

```text
virtio escape surface:
- Shared virtqueue descriptor and indirect-descriptor handling
- Per-device backends: net, block, scsi, gpu, balloon, fs
- Feature negotiation and configuration-space handling
```

## Exploitation notes

- virtio is the broadest surface because essentially every guest enables it; virtio-gpu and virtio-net have been particularly productive.
- When the backend runs in the kernel (vhost-net, vhost-scsi), a bug lands in the host kernel rather than the QEMU process, which is higher impact.
- Reachability is near-universal, so the device type on the guest determines which backend is in play.

## References

- [virtio specification](https://docs.oasis-open.org/virtio/virtio/v1.2/virtio-v1.2.html)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
