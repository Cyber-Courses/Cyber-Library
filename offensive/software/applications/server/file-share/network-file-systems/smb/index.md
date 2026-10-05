---
title: "SMB: attacking Windows file shares"
description: "Attacking SMB, the dominant enterprise file-sharing protocol: null-session and guest enumeration, share listing and permissions, signing and NTLM relay, writable-share poisoning that coerces or executes on other users, looting readable shares for secrets, and the named protocol remote code execution flaws."
keywords:
  - SMB
  - CIFS
  - null session
  - share enumeration
  - NTLM relay
---

# SMB

SMB (Server Message Block) is how Windows environments share files, and it is the richest file-share target. Attacks span reaching it without credentials (null session, guest), mapping and reading shares, abusing missing signing to relay authentication, poisoning writable shares to coerce or execute code on other users, looting readable shares for credentials, and the protocol-level remote code execution flaws in the SMB server itself.

## Subtopics

- **[Share enumeration](share-enumeration.md)**: listing shares and their permissions.
- **[Null session](null-session.md)**: anonymous access to IPC and enumeration.
- **[Signing and relay](signing-and-relay.md)**: missing signing and NTLM relay.
- **[Writable share poisoning](writable-share-poisoning/index.md)**: planting files that act on other users.
- **[Loot and sensitive files](loot-and-sensitive-files/index.md)**: harvesting secrets from readable shares.
- **[Known SMB exploits](known-smb-exploits.md)**: protocol-level remote code execution.

## References

- [HackTricks: pentesting SMB](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
- [NetExec (nxc) documentation](https://www.netexec.wiki/)
