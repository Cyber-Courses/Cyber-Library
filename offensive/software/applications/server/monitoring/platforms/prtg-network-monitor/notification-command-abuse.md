---
title: "Notification command abuse: SYSTEM RCE through PRTG notifications"
order: 2
description: "PRTG notifications can execute an external program or script when triggered, and the command parameter is attacker-controlled by an authenticated administrator. A crafted parameter injects OS commands that run as the PRTG core service account, Local System, so an admin creates or edits an Execute Program notification and triggers it to gain SYSTEM code execution on the server."
keywords:
  - prtg notification
  - execute program
  - command injection
  - system
  - rce
---

# Notification command abuse

PRTG's notification system can run an external program or script as an action (the "Execute Program" / demo notification type), and the parameters passed to it are controlled by an authenticated administrator through the web interface. Because those parameters are placed into a command executed by the PRTG probe/core, a crafted parameter containing OS command syntax injects additional commands, and since the PRTG core service runs as Local System on Windows, the injected commands execute as SYSTEM. The attack is therefore: as an administrator, create or edit a notification of the execute-program type with a parameter carrying a payload, then trigger the notification (manually test it, or make a sensor enter a state that fires it), and the payload runs as SYSTEM on the monitoring server.

```bash
# authenticated (prtgadmin). Create/edit an "Execute Program" notification whose
# parameter field carries a command payload, then trigger it. The parameter is run
# by the core (as SYSTEM); injecting PowerShell/command syntax executes it, e.g.:
#   parameter:  test.txt;net user pwn P@ss1 /add;net localgroup administrators pwn /add
# the historic auth command-injection (CVE class) abused exactly this notification
# parameter to run commands as SYSTEM. Tools/metasploit automate create+trigger.
curl -sk -b cj 'https://<target>:8080/editnotification.htm' --data '<notification-with-payload>'
curl -sk -b cj 'https://<target>:8080/api/notificationtest.htm?id=<notifid>'   # trigger
```

## Exploitation notes

- The precondition is just an administrator session: the notification parameter is admin-controlled and reaches a command run as SYSTEM, so admin equals SYSTEM RCE.
- Trigger the notification to fire the payload: PRTG offers a test/execute action for notifications, or a sensor state change invokes it; the test action is the simplest.
- Execution is SYSTEM on the Windows monitoring server, which also stores the credentials PRTG uses to monitor devices (and is often domain-joined), so it is a strong pivot; add a local admin or run a payload directly.
- This is the canonical PRTG RCE; the specific injection and any required encoding are version-dependent, see [Known exploits](known-exploits.md).

## References

- [PRTG: notifications](https://www.paessler.com/manuals/prtg/notifications_settings)
- [HackTricks: PRTG RCE](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/prtg)
