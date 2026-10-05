---
title: "Clipboard redirection: capturing data through the RDP clipboard channel"
description: "RDP's clipboard-redirection virtual channel syncs the clipboard between client and server for convenience. An attacker who controls either end monitors and captures the synced clipboard, which routinely carries passwords copied from managers, tokens, and sensitive text, and can inject content, turning a shared clipboard into a data-theft and influence channel."
keywords:
  - clipboard redirection
  - cliprdr
  - rdp channel
  - data theft
  - clipboard monitoring
---

# Clipboard redirection

RDP synchronises the clipboard between the client and the remote session over the `cliprdr` virtual channel, so a copy on one side is available on the other. That convenience is a data channel an attacker abuses from whichever end they control. Monitoring the synced clipboard captures what users copy, which very often includes passwords pasted from a password manager, API tokens, one-time codes, and sensitive text, and the channel is bidirectional, so an attacker can also inject clipboard content to influence what a user pastes. On a compromised terminal server, or a malicious RDP endpoint a user connects to, the clipboard becomes a theft and manipulation surface.

```bash
# on a controlled RDP host/session, poll the clipboard for secrets
# (PowerShell in the session)
while ($true) { Get-Clipboard; Start-Sleep -Seconds 2 }     # capture what users copy
# a malicious RDP server/endpoint can likewise log the cliprdr channel contents
# injection: set the clipboard to attacker content the user may paste
Set-Clipboard -Value 'attacker-controlled'
```

## Exploitation notes

- Password managers encourage copy-paste of credentials, so a monitored RDP clipboard is a reliable credential source; poll it on a host where users have sessions, or on a malicious endpoint they connect to.
- Injection is the subtler abuse: replacing clipboard content (for example swapping a pasted command or a cryptocurrency address) manipulates what the user acts on.
- This requires control of one end of the session; on a compromised terminal server you read every user's clipboard, and a rogue RDP destination reads the connecting client's.
- Clipboard redirection is enabled by default in many configurations; its presence is part of the session's device-redirection surface alongside [drive redirection](drive-redirection.md).

## References

- [MS-RDPECLIP (clipboard virtual channel)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpeclip/)
- [HackTricks: RDP redirection](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
