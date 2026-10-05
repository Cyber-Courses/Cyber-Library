---
title: "Network file systems: attacking shared-filesystem protocols"
description: "Network file systems export filesystems over the network for mounting by clients: SMB on Windows, NFS on Unix, and AFP on Apple and NAS devices. Each trusts the client to varying degrees, SMB through sessions and NTLM, NFS through client-asserted UIDs, AFP through guest and appliance accounts, and each exposes enumeration, access abuse, looting, and implementation flaws."
keywords:
  - network file system
  - smb
  - nfs
  - afp
  - file sharing
---

# Network file systems

A network file system exports a server's filesystem so clients can mount and use it as if local. The three that matter are SMB (the Windows standard, also Samba), NFS (the Unix standard), and AFP (Apple's protocol, now mostly on NAS devices via Netatalk). They differ sharply in how much they trust the client: SMB establishes sessions and uses NTLM/Kerberos, so its surface includes relay and session abuse; NFS with AUTH_SYS trusts the UID the client asserts, so impersonation is trivial; AFP leans on guest access and appliance accounts. Across all three the recurring moves are enumerating exports and access, abusing weak access models, looting readable data, and exploiting the server implementation.

## Subtopics

- **[SMB](smb/index.md)**: the Windows file-sharing protocol, sessions, shares, relay, looting, and poisoning.
- **[NFS](nfs/index.md)**: the Unix network file system and its AUTH_SYS trust model.
- **[AFP](afp/index.md)**: the Apple Filing Protocol and Netatalk.

## References

- [MS-SMB2](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/)
- [NFS (nfs(5), RFC 8881)](https://man7.org/linux/man-pages/man5/nfs.5.html)
- [Netatalk / AFP](https://netatalk.io/)
