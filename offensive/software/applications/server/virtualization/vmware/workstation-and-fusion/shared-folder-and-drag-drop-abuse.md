---
title: "Shared folder and drag-drop abuse: escaping through desktop integration"
order: 2
description: "Workstation and Fusion integration features, shared folders (HGFS), drag-and-drop, and copy-paste, are implemented as GuestRPC handlers in vmware-vmx that parse complex guest-supplied requests. Shared folders also expose host paths into the guest. Both the path handling and the RPC parsing have repeatedly yielded guest-to-host escapes and host file access."
keywords:
  - hgfs
  - shared folders
  - drag and drop
  - copy paste
  - guestrpc
---

# Shared folder and drag-drop abuse

The conveniences that make desktop VMs pleasant, shared folders, drag-and-drop, and copy-paste, are also an escape surface. Shared folders use the Host-Guest File System (HGFS) protocol, and drag-and-drop and copy-paste (DnD/CP) use their own protocols, all carried over GuestRPC and parsed in the `vmware-vmx` process. Two problems follow. The protocol handlers parse complex, nested, guest-controlled requests in the host process, a recurring source of memory-corruption escapes. And shared folders expose host directories into the guest, so path-traversal in the HGFS path handling reaches host files outside the shared directory.

## HGFS path handling

```bash
# when a shared folder is enabled, it is mounted in the guest via vmhgfs/FUSE
mount | grep -i hgfs; ls /mnt/hgfs/ 2>/dev/null
# the HGFS protocol resolves guest-supplied paths against the shared host directory;
# a traversal in request path handling (../ sequences, symlink tricks, name parsing)
# can escape the shared root and read/write host files outside it
```

## DnD/CP and HGFS RPC parsing

```bash
# drag-and-drop, copy-paste, and HGFS requests travel over GuestRPC (the backdoor)
# the vmx-side handlers parse length-prefixed, nested structures; a length or index
# the handler trusts, or a use-after-free on a transfer object, corrupts vmware-vmx.
# a guest speaks these RPCs directly (see Backdoor and VMCI) without the Tools UI.
```

The DnD and copy-paste version negotiation and the transfer-object lifecycle have been particularly productive, with use-after-free and out-of-bounds bugs in how the handlers track in-progress transfers.

## Exploitation notes

- Two distinct impacts: HGFS path handling can give host file read/write within and beyond the shared folder, while the RPC parsing bugs give full code execution in `vmware-vmx`.
- These handlers are reachable through GuestRPC directly, so the attack does not depend on using the Tools GUI; a guest driver issues the crafted RPCs, see [Backdoor and VMCI](../esxi/guest-to-host-escape/backdoor-and-vmci.md).
- The feature must be enabled for the richest surface (a configured shared folder, DnD/CP allowed), which is common on desktop VMs; check the VM settings or the guest mounts.
- The same HGFS/DnD code exists on ESXi, so these bugs often cross over; this surface is simply more exposed on desktop products.

## References

- [open-vm-tools HGFS implementation](https://github.com/vmware/open-vm-tools)
- [Zero Day Initiative: VMware DnD/HGFS research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
