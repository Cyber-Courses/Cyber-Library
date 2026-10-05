---
title: "SMB: attacking the Windows file-sharing protocol"
description: "SMB (Server Message Block) serves Windows file shares over TCP 445 and underpins much of a Windows network. The offensive surface is session establishment (null and guest sessions), share and permission enumeration, looting readable shares and poisoning writable ones, the message-signing and NTLM-relay weaknesses, and the protocol's own remote code execution flaws."
keywords:
  - smb
  - port 445
  - ntlm relay
  - share enumeration
  - null session
---

# SMB

SMB is the Server Message Block protocol, serving Windows (and Samba) file shares on TCP 445, and it is central to Windows environments: file shares, the `SYSVOL`/`NETLOGON` domain shares, named pipes for RPC, and administrative shares (`C$`, `ADMIN$`) all ride on it. Offensively it is one of the richest services on a network. The surface spans how a session is established (anonymous null sessions, guest access, or credentials), enumerating shares and the per-identity access to them, looting readable shares and poisoning writable ones to capture credentials or plant code, the message-signing and NTLM-relay weaknesses that turn authentication into lateral movement, and the protocol implementation's own remote code execution bugs.

```bash
# fingerprint and reach the service
nmap -p445 --script smb-protocols,smb2-security-mode <target>
nxc smb <target>                              # OS, domain, signing, SMBv1 status
```

## Subtopics

- **[Share enumeration](share-enumeration.md)**: listing shares and per-identity access.
- **[Null session](null-session.md)**: anonymous access to shares and RPC.
- **[Signing and relay](signing-and-relay.md)**: SMB signing weaknesses and NTLM relay.
- **[Loot and sensitive files](loot-and-sensitive-files/index.md)**: mining readable shares.
- **[Writable share poisoning](writable-share-poisoning/index.md)**: planting payloads and coercion files.
- **[Known SMB exploits](known-smb-exploits.md)**: the protocol's remote code execution flaws.

## References

- [MS-SMB2 protocol specification](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/)
- [NetExec: SMB protocol](https://www.netexec.wiki/smb-protocol)
- [Impacket](https://github.com/fortra/impacket)
