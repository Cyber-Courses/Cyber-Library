---
title: "Session abuse: hijacking, shadowing, and device redirection"
description: "Beyond logging in, RDP sessions themselves are abusable. An administrator or SYSTEM-level attacker on a host takes over other users' disconnected or active sessions without their password, shadows sessions to watch them live, and the device-redirection channels (clipboard, drives) leak data between client and server. These turn host access into lateral credential and data capture."
keywords:
  - rdp session
  - session hijacking
  - shadowing
  - clipboard
  - drive redirection
---

# Session abuse

RDP sessions are an attack surface in their own right, exploited once an attacker already has privileged access to a terminal server or multi-user host. Windows lets a sufficiently privileged account connect to another user's session: hijacking takes over a disconnected or even active session without the victim's password, and shadowing observes a live session. Separately, RDP's device-redirection virtual channels, clipboard and drive mapping, move data between the client and server and leak it to whoever controls either end. These techniques convert host access into capturing other users' sessions, credentials, and data.

```bash
# enumerate sessions on a host you have privileged access to
query user            # or: qwinsta   (session IDs, states: Active/Disc)
```

## Subtopics

- **[Session hijacking](session-hijacking.md)**: taking over another user's session without their password.
- **[Session shadowing](session-shadowing.md)**: observing a live session.
- **[Clipboard redirection](clipboard-redirection.md)**: capturing data via the clipboard channel.
- **[Drive redirection](drive-redirection.md)**: reaching mapped client drives.

## References

- [Microsoft: Remote Desktop Services sessions](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
- [HackTricks: RDP session abuse](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
