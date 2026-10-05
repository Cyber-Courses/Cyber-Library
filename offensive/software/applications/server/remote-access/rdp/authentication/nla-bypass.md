---
title: "NLA bypass: Network Level Authentication considerations and misconfigurations"
description: "Network Level Authentication makes an RDP client authenticate via CredSSP before a session is created, which blunts pre-auth exploitation and unauthenticated logon-screen access. The offensive angle is not breaking CredSSP but exploiting hosts where NLA is disabled or not enforced, exposing the pre-auth surface and the logon screen, and chaining stolen credentials through CredSSP."
keywords:
  - nla
  - credssp
  - network level authentication
  - pre-auth surface
  - misconfiguration
---

# NLA bypass

Network Level Authentication (NLA) requires an RDP client to authenticate through CredSSP before the server creates a session. It is a meaningful hardening: it removes unauthenticated access to the logon screen and the pre-session code paths where the wormable RDP bugs lived. The realistic offensive angle is not cryptographically breaking CredSSP but targeting where NLA is absent or not enforced. Many hosts, especially older or internally-facing ones, run with NLA disabled, which re-exposes the pre-authentication surface and the interactive logon screen to unauthenticated clients; and NLA does not stop credential guessing or replay, it just moves the check into CredSSP.

```bash
# determine whether NLA is required (the decisive fact)
nmap -p3389 --script rdp-enum-encryption <target>   # reports NLA supported vs required
# if NLA is NOT required, connect to the logon surface without credentials
xfreerdp /v:<target> /cert:ignore                   # reaches logon when NLA is off
# if NLA IS required, credentials are checked by CredSSP first -> use valid/stuffed creds
xfreerdp /v:<target> /u:'DOMAIN\user' /p:'pass' /cert:ignore
```

## Exploitation notes

- The win is finding NLA off, not defeating NLA: an NLA-disabled host exposes the pre-auth protocol surface ([BlueKeep](../pre-authentication-flaws/bluekeep.md)/[DejaBlue](../pre-authentication-flaws/dejablue.md)) and the logon screen to unauthenticated connections, which NLA-required hosts do not.
- With NLA required, attacks shift to obtaining valid credentials (brute force, spraying, stuffing) that satisfy CredSSP; NLA changes where credentials are verified, not whether they can be guessed or replayed.
- CredSSP has had its own vulnerabilities historically; those are protocol-implementation bugs to match by patch level, separate from the NLA-off misconfiguration case.
- Enumerate NLA status first ([security settings](../enumeration/security-settings.md)); it routes the entire RDP attack.

## References

- [Microsoft: Network Level Authentication and CredSSP](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
- [HackTricks: RDP NLA](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
