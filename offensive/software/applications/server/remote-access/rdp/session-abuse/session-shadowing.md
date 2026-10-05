---
title: "Session shadowing: observing a live RDP session"
description: "RDP session shadowing lets a privileged account view, and optionally control, another user's active session. Intended for support, it is abused by an attacker with admin rights on a host to watch a target's live session, capturing what they type and see, and with control enabled, to interact as them, all governed by a policy that can permit shadowing without consent."
keywords:
  - session shadowing
  - rdp shadow
  - mstsc shadow
  - surveillance
  - terminal services
---

# Session shadowing

Shadowing is the Remote Desktop Services feature that lets one user view or control another's active session, built for helpdesk support. An attacker with administrative rights on a terminal server abuses it as live surveillance and interaction: viewing a target's session captures everything they see and type (credentials entered, documents opened, commands run), and with control enabled the attacker drives the session as that user. A Group Policy setting governs whether shadowing requires the user's consent; where it is set to allow view or control without consent, the target is unaware.

```bash
# enumerate sessions, then shadow by ID
query user            # or qwinsta: find the target session ID
# shadow from an admin context (view or control per policy)
mstsc /shadow:<SESSION_ID> /v:<target> /control /noConsentPrompt
# the /noConsentPrompt and control behaviour depend on the Shadow policy on the host
```

## Exploitation notes

- The consent behaviour is set by the "Set rules for remote control of Remote Desktop Services user sessions" policy; where it permits view/control without consent, shadowing is covert, otherwise the user is prompted.
- View-only shadowing is quiet surveillance, capturing credentials and sensitive content as the user works; control turns it into acting as the user within their live session.
- Like hijacking, this requires existing administrative rights on the host, so it is a post-access technique on terminal servers and jump hosts; it differs from [session hijacking](session-hijacking.md) in that it observes/joins rather than taking the session over.
- Use it to harvest credentials a target types and to catch privileged actions in progress, then pivot with what is captured.

## References

- [Microsoft: RDS session remote control (shadowing)](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
- [HackTricks: RDP shadowing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
