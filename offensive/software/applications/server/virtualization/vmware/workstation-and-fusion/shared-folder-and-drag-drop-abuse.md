---
title: "Shared folder and drag-drop abuse: reaching the host from a guest"
description: "Abusing VMware Workstation and Fusion integration features, Shared Folders (HGFS), drag-and-drop, and clipboard sharing, to read and write host files or trigger host-side parsing bugs from inside a guest when these conveniences are left enabled."
keywords:
  - Shared Folders
  - HGFS
  - drag and drop
  - clipboard
  - VMware Tools
---

# Shared folder and drag-drop abuse

VMware Tools adds host integration: Shared Folders (HGFS) expose host directories to the guest, and drag-and-drop and clipboard sharing move data across the boundary. When enabled, Shared Folders is a direct host file read and write primitive from the guest, and the drag-and-drop and HGFS request handlers have themselves been guest-to-host code-execution bugs.

```bash
# Inside a Linux guest with Shared Folders enabled
ls /mnt/hgfs/                                   # host directories exposed to the guest
vmware-hgfsclient                               # list configured shares
# Writing into a shared folder reaches the host filesystem directly
```

## Exploitation notes

- Shared Folders is often enabled in analysis VMs for convenience; it turns a guest foothold into host file access with no exploit.
- The HGFS and drag-and-drop request parsers have produced memory-corruption escapes in their own right, reachable only when the feature is on.
- Disabling Shared Folders and drag-and-drop removes this surface, so its presence is the precondition to check first.

## References

- [VMware: using Shared Folders](https://docs.vmware.com/en/VMware-Workstation-Pro/index.html)
- [Zero Day Initiative: VMware research](https://www.zerodayinitiative.com/blog)
