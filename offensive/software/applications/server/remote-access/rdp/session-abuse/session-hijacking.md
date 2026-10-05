---
title: "Session hijacking: taking over an RDP session without a password"
description: "On a Windows host, a SYSTEM-level attacker uses tscon to connect to another user's RDP session, disconnected or active, and attach it to their own, without knowing the victim's password. This hands the victim's full interactive session and privileges, a powerful lateral-movement and privilege-escalation technique on terminal servers and jump hosts."
keywords:
  - session hijacking
  - tscon
  - rdp
  - terminal services
  - privilege escalation
---

# Session hijacking

Windows Remote Desktop Services allows a privileged account to connect one session to another, and run as SYSTEM this requires no knowledge of the target user's password. The `tscon` command attaches a specified session to the current one; executed as SYSTEM (for example via a service), it moves another user's session, disconnected or active, onto the attacker's connection, giving their full interactive desktop and privileges. On terminal servers, Citrix hosts, and jump boxes where many users have sessions, this is a direct route to impersonating a more privileged user, including domain admins who left disconnected sessions.

```bash
# enumerate sessions and find a higher-privileged target
query user            # ID, STATE (Active/Disc), USERNAME
# hijack as SYSTEM: create a service that runs tscon to attach the target session
#   (run as SYSTEM so no password is needed)
sc create hijack binpath= "cmd /k tscon <TARGET_SESSION_ID> /dest:<YOUR_SESSION>"
sc start hijack
# or with PsExec -s to get SYSTEM, then: tscon <id> /dest:console
```

## Exploitation notes

- The no-password property holds only when running as SYSTEM; from an admin shell, elevate to SYSTEM first (a service, `PsExec -s`, or a scheduled task) and then `tscon` attaches the target session with no credential prompt.
- Target disconnected sessions of privileged users first: a domain admin's disconnected session on a jump host is a direct path to their rights, and hijacking it is quiet (the user is not actively watching).
- Hijacking an active session will disconnect or surprise the live user, so it is noisier; disconnected sessions are preferred.
- This is lateral movement and privilege escalation local to the host; it requires existing admin/SYSTEM on the terminal server, so it chains after initial RDP or other access.

## Tools

- [tscon / query (built-in)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tscon)
- [PsExec (SYSTEM)](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec)

## References

- [Microsoft: tscon](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tscon)
- [HackTricks: RDP session hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
