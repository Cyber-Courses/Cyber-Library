---
title: "Shared folder and clipboard abuse: reaching the host from VirtualBox"
description: "Abusing VirtualBox Guest Additions integration, Shared Folders, the shared clipboard, and drag-and-drop, to read and write host files or move data across the boundary from inside a guest when these features are enabled."
keywords:
  - VirtualBox shared folders
  - Guest Additions
  - clipboard
  - drag and drop
  - host access
---

# Shared folder and clipboard abuse

VirtualBox Guest Additions add Shared Folders, a shared clipboard, and drag-and-drop. Shared Folders mounts host directories into the guest, a direct host file read and write primitive, and the clipboard and drag-and-drop channels move data across the boundary. These are conveniences commonly left on in lab and analysis VMs.

```bash
# Inside a Linux guest with Guest Additions and a shared folder
mount | grep vboxsf
ls /media/sf_<share>/                           # host directory exposed to the guest
# Writing here reaches the host filesystem
```

## Exploitation notes

- Shared Folders turns a guest foothold into host file access with no exploit; writing into a shared path can plant a payload on the host.
- The shared clipboard leaks data in both directions and has been an information-disclosure and parsing-bug surface.
- These features require Guest Additions and explicit configuration, so their presence is the precondition to check.

## References

- [VirtualBox: shared folders](https://www.virtualbox.org/manual/ch04.html#sharedfolders)
- [VirtualBox: Guest Additions](https://www.virtualbox.org/manual/ch04.html)
