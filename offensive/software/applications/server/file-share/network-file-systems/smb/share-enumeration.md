---
title: "Share enumeration: listing SMB shares and permissions"
description: "Enumerating the shares an SMB server exposes and the access the current identity has to each, with a null, guest, or valid session, to find readable shares to loot and writable shares to abuse."
keywords:
  - SMB share enumeration
  - smbclient
  - NetExec
  - share permissions
  - spider
---

# Share enumeration

The first step against an SMB server is listing its shares and, crucially, the read and write access the current identity has to each. A null, guest, or valid session all enumerate shares; the goal is to separate readable shares (loot) from writable ones (poisoning and planting).

```bash
# List shares and per-share read/write access (null, guest, or creds)
nxc smb <target> -u '' -p '' --shares
nxc smb <target> -u user -p pass --shares
smbclient -L //<target>/ -N                 # list shares anonymously
# Spider a share for files
nxc smb <target> -u user -p pass -M spider_plus
```

## Exploitation notes

- NetExec's `--shares` marks READ and WRITE per share, which immediately flags loot and poisoning targets.
- Non-default shares (not `C$`, `ADMIN$`, `IPC$`, `SYSVOL`, `NETLOGON`) are usually the interesting business data.
- A writable share leads to [Writable share poisoning](writable-share-poisoning/index.md); a readable one to [Loot and sensitive files](loot-and-sensitive-files/index.md).

## References

- [NetExec: SMB protocol](https://www.netexec.wiki/smb-protocol/enumeration)
- [smbclient manual](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)
