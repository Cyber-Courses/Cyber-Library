---
title: "Signing and relay: missing SMB signing and NTLM relay"
description: "Identifying SMB servers that do not require message signing, and relaying captured or coerced NTLM authentication to them to act as the victim against the file share, the file-sharing view of the broader NTLM relay attack covered under Directory."
keywords:
  - SMB signing
  - NTLM relay
  - ntlmrelayx
  - coercion
  - message signing
---

# Signing and relay

SMB message signing prevents an attacker from relaying someone else's authentication to a server. Where signing is not required (the default on member servers), a captured or coerced NTLM authentication can be relayed to the file server and the attacker acts as that user, reaching the shares the victim can. This is the file-sharing face of NTLM relay; the capture and coercion chain is covered in depth under Directory.

```bash
# Find servers where SMB signing is NOT required (relay targets)
nxc smb <subnet> --gen-relay-list targets.txt
# Relay coerced/captured auth to a share server
ntlmrelayx.py -tf targets.txt -smb2support
```

## Exploitation notes

- A relay to a file server yields that victim's share access, including reading or writing files as a privileged user.
- Relaying a computer or admin account to a share can enable file write that leads to execution (service binaries, startup scripts).
- The capture side (Responder, coercion like PetitPotam) and the full relay matrix live under the Directory area and are cross-referenced.

## References

- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
- [NetExec: generating relay lists](https://www.netexec.wiki/smb-protocol/enumeration)
