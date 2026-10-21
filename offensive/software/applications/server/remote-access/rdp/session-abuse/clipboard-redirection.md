---
title: "Clipboard redirection: capturing data through the RDP clipboard channel"
description: "RDP's clipboard-redirection virtual channel synchronizes a client's clipboard with a specific remote session. An attacker executing within that session, or controlling its client endpoint, can monitor or replace redirected clipboard content when the channel is enabled."
keywords:
  - clipboard redirection
  - cliprdr
  - rdp channel
  - data theft
  - clipboard monitoring
---

# Clipboard redirection

RDP synchronizes the clipboard between one client connection and its remote session over the `cliprdr` virtual channel. Code executing in that interactive session can read or replace its clipboard, potentially exposing passwords, API tokens, one-time codes, or sensitive text copied through the channel. The scope matters: clipboard redirection is session-bound. Compromising a terminal server does not make every user's redirected clipboard visible to a single process; the attacker needs execution in, or control of, each target session. A malicious RDP endpoint can likewise observe the clipboard redirected by the client connected to it.

```bash
# run inside the specific interactive session whose clipboard is being assessed
while ($true) { Get-Clipboard; Start-Sleep -Seconds 2 }
# a malicious RDP server/endpoint can likewise log the cliprdr channel contents
# injection: set the clipboard to attacker content the user may paste
Set-Clipboard -Value 'attacker-controlled'
```

## Exploitation notes

- Clipboard contents can include credentials or tokens, but `Get-Clipboard` observes the clipboard of the session in which the process runs, not every session on the host.
- Injection is the subtler abuse: replacing clipboard content (for example swapping a pasted command or a cryptocurrency address) manipulates what the user acts on.
- This requires control of the relevant session or client endpoint. To target another logged-on user on a session host, first obtain execution in or control of that user's session.
- Clipboard redirection is enabled by default in many configurations; its presence is part of the session's device-redirection surface alongside [drive redirection](drive-redirection.md).

## References

- [MS-RDPECLIP (clipboard virtual channel)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpeclip/)
- [HackTricks: RDP redirection](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
