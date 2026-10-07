---
title: "Shared folder and clipboard abuse: escaping Parallels through Tools integration"
order: 2
description: "Parallels Tools provides shared folders, a shared clipboard, and drag-and-drop between the guest and the macOS host over Parallels' guest-host communication channels. The host-side handlers parse guest-supplied integration requests, and shared folders expose host paths, so both the request parsing and the shared-folder path handling have produced guest-to-host escapes and host file access."
keywords:
  - parallels tools
  - shared folders
  - clipboard
  - drag and drop
  - integration
---

# Shared folder and clipboard abuse

Parallels Tools, the guest integration package, provides shared folders, a shared clipboard, and drag-and-drop between the guest and the macOS host. These travel over Parallels' guest-host communication channels, and the host-side handlers parse the guest's integration requests in the Parallels processes. As with VMware's GuestRPC/HGFS and VirtualBox's HGCM, this is a prominent escape surface: the request parsing has yielded memory-corruption escapes, and shared-folder path handling has allowed host file access beyond the shared directory.

## The surface

```text
Integration surface (Parallels Tools channels):
- shared folders: a host directory exposed into the guest; path resolution runs
  host-side, so traversal/symlink/name-parsing flaws reach host files outside the share
- shared clipboard and drag-and-drop: the host parses guest-supplied formats, sizes,
  and transfer objects; a trusted length/size or a transfer-object lifecycle bug
  (use-after-free) corrupts the host process
- the Tools control channel: integration requests parsed host-side with the same classes
```

```bash
# Parallels Tools and the shared-folder mount (Linux guest)
lsmod 2>/dev/null | grep -i prl
mount 2>/dev/null | grep -i prl_fs       # shared folders via the Parallels filesystem
ls /media/psf 2>/dev/null                 # "psf" = Parallels Shared Folders mount point
```

## Exploitation notes

- Two impacts, as on the comparable hypervisors: request parsing bugs give code execution in the host Parallels process, while shared-folder path handling can give host file read/write within and beyond the shared directory.
- Drag-and-drop and clipboard transfer-object handling are frequent loci (version negotiation, in-progress transfer tracking), the Parallels analogue of VMware DnD/CP and VirtualBox HGCM bugs.
- The feature must be enabled for the richest surface (a configured shared folder, clipboard/DnD allowed), common on desktop VMs; check the `psf` mount and Tools modules.
- Drive the channels from a controlled guest Tools client; the device escapes are under [Guest-to-host escape](guest-to-host-escape.md). Version-specific.

## References

- [Parallels Desktop: shared folders and Tools](https://www.parallels.com/products/desktop/)
- [Zero Day Initiative: Parallels Tools research](https://www.zerodayinitiative.com/blog)
