---
title: "Share enumeration: listing SMB shares and permissions"
description: "Enumerating the shares an SMB server exposes and the read/write access the current identity has to each, with a null, guest, or valid session. The access map separates readable shares to loot from writable shares to poison, and the share comments and names reveal the server's role and data, directing every subsequent SMB action."
keywords:
  - smb share enumeration
  - smbclient
  - netexec
  - share permissions
  - spider
---

# Share enumeration

Listing an SMB server's shares and the current identity's access to each is the first move and the one that directs all the others. A share is a named export of a directory (or a special resource like `IPC$` for named pipes), and access is governed by both share-level permissions and the underlying NTFS permissions, so the effective read/write for a given identity must be tested, not assumed. The output separates readable shares, which become loot, from writable shares, which become poisoning and planting targets, and the share names and comments often reveal the server's role (file server, backup target, deployment share).

```bash
# list shares and per-share READ/WRITE for the identity (null, guest, or creds)
nxc smb <target> -u '' -p '' --shares          # anonymous
nxc smb <target> -u user -p 'pass' --shares     # authenticated
nxc smb <target> -u user -H <nthash> --shares   # pass-the-hash
smbclient -L //<target>/ -N                      # list anonymously (no access map)
# connect to a specific share to confirm access and browse
smbclient //<target>/<share> -U 'DOM\user%pass'
#   smb: \> ls ; recurse ON ; prompt OFF ; mget *
```

Read the NetExec output column by column: `READ` and `WRITE` flags per share are the decision, and the share list separates the default administrative shares from business data.

## Spider and map content

```bash
# recursively list files across shares to find loot without downloading everything
nxc smb <target> -u user -p 'pass' -M spider_plus          # writes a JSON inventory
nxc smb <target> -u user -p 'pass' -M spider_plus -o DOWNLOAD_FLAG=True READ_ONLY=False
# manual recursive listing of one share
smbclient //<target>/<share> -U 'user%pass' -c 'recurse ON; ls'
```

## Exploitation notes

- The READ/WRITE map is the whole point of this step: `--shares` marks each, so a writable non-default share flags a [poisoning](writable-share-poisoning/index.md) target and a readable one a [looting](loot-and-sensitive-files/index.md) target.
- Ignore the default shares (`C$`, `ADMIN$`, `IPC$`, `SYSVOL`, `NETLOGON`) for data, but note `C$`/`ADMIN$` access implies administrative rights (a different, higher-value finding), and `SYSVOL` is readable by any domain user and holds Group Policy (and sometimes credentials).
- Effective access depends on both share and NTFS permissions; a share listed without a flag for a null session may still be readable with guest or valid credentials, so re-run as each identity you obtain.
- `spider_plus` produces a file inventory per share to triage offline before pulling data, which is quieter than mass download.

## Tools

- [NetExec](https://github.com/Pennyw0rth/NetExec)
- [smbclient (Samba)](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)

## References

- [NetExec: SMB enumeration](https://www.netexec.wiki/smb-protocol/enumeration)
- [MS-SMB2: TREE_CONNECT](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/)
