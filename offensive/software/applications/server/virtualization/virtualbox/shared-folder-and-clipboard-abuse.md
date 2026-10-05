---
title: "Shared folder and clipboard abuse: escaping VirtualBox through Guest Additions"
description: "VirtualBox Guest Additions provide shared folders, a shared clipboard, and drag-and-drop over the Host-Guest Communication Manager (HGCM). The host-side HGCM services parse guest-supplied requests in the VM process, and shared folders expose host paths, so both the HGCM parsing and the shared-folder path handling have yielded guest-to-host escapes and host file access."
keywords:
  - guest additions
  - hgcm
  - shared folders
  - clipboard
  - vboxsf
---

# Shared folder and clipboard abuse

The VirtualBox Guest Additions add host integration, shared folders (`vboxsf`), a shared clipboard, and drag-and-drop, carried over the Host-Guest Communication Manager (HGCM), a request/response channel between the guest and the host VM process. The HGCM services (shared folders, clipboard, drag-and-drop, guest properties, and more) parse guest-supplied requests in the host process, and shared folders additionally expose host directories into the guest. Both are escape surfaces: HGCM request parsing has produced memory-corruption escapes, and shared-folder path handling has allowed host file access beyond the shared directory.

## The HGCM surface

```c
// a guest issues HGCM calls to a named service with typed parameters (pointers to
// guest buffers with lengths, 32/64-bit values). The host service parses them.
//  - a parameter length/count the service trusts for a copy -> OOB in the VM process
//  - a service-specific request (shared-folder op, clipboard format) with a crafted
//    size or index the handler does not bound
// services: VBoxSharedFolders, VBoxSharedClipboard, VBoxDragAndDrop, VBoxGuestProps
```

```bash
# Guest Additions and the shared-folder mount
lsmod | grep -i vboxguest; mount | grep -i vboxsf
VBoxControl guestproperty enumerate 2>/dev/null   # the guest-properties HGCM service
```

## Shared-folder path handling

```bash
# a shared folder maps a host directory; its path resolution runs host-side
ls /media/sf_* /mnt/*sf* 2>/dev/null
# path-handling flaws (traversal, symlink, name parsing) in the shared-folder service
# can reach host files outside the shared root, or mishandle a crafted request
```

## Exploitation notes

- Two impacts: HGCM request parsing bugs give code execution in the VM process, while shared-folder path handling can give host file read/write within and beyond the shared directory.
- The HGCM services are reachable through the Guest Additions interface from a controlled guest; an attacker issues crafted HGCM calls directly rather than using the normal integration UI.
- The feature must be enabled for the richest surface (a configured shared folder, clipboard/DnD allowed), common on desktop VMs; check the mounts and guest properties.
- These are a staple VirtualBox Pwn2Own surface; the device escapes are under [Guest-to-host escape](guest-to-host-escape/index.md).

## References

- [VirtualBox manual: shared folders and Guest Additions](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox HGCM research](https://www.zerodayinitiative.com/blog)
