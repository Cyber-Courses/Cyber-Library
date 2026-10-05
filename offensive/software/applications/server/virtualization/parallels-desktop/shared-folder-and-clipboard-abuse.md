---
title: "Shared folder and clipboard abuse: reaching macOS from Parallels"
description: "Abusing Parallels Desktop integration, Shared Folders, Shared Profile, the shared clipboard, and drag-and-drop, to read and write macOS host files or move data across the boundary from inside a guest when these features are enabled."
keywords:
  - Parallels shared folders
  - Shared Profile
  - clipboard
  - drag and drop
  - Parallels Tools
---

# Shared folder and clipboard abuse

Parallels integration, provided by Parallels Tools, shares the macOS home folders with the guest (Shared Folders and Shared Profile), and syncs the clipboard and drag-and-drop. When enabled, Shared Folders is a direct read and write path into the macOS filesystem from the guest, and the clipboard and drag-and-drop channels move data and files across the boundary.

```bash
# Inside a Linux guest with Parallels Tools and sharing enabled
ls /media/psf/                              # macOS host folders exposed to the guest
ls /media/psf/Home/                          # the user's macOS home (Shared Profile)
# Writing here reaches the macOS filesystem
```

## Exploitation notes

- Shared Profile maps the user's entire macOS home into the guest, turning a guest foothold into broad host file access with no exploit.
- Writing into a shared path can plant a macOS launch agent or payload for execution on the host.
- These features require Parallels Tools and explicit sharing settings, so their presence is the precondition to check.

## References

- [Parallels Desktop: sharing between macOS and the guest](https://www.parallels.com/products/desktop/resources/)
- [Parallels Tools overview](https://kb.parallels.com/)
