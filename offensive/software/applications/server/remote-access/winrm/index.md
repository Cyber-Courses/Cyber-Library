---
title: "WinRM: attacking Windows Remote Management"
order: 8
description: "WinRM implements WS-Management on TCP 5985 (HTTP) and 5986 (HTTPS), providing remote command execution on Windows through PowerShell Remoting. The surface is service enumeration, authentication with passwords or NTLM hashes (pass-the-hash), and command execution: valid credentials or a hash for a Remote Management user yield an interactive shell, typically with administrative rights."
keywords:
  - winrm
  - ws-management
  - powershell remoting
  - evil-winrm
  - port 5985
---

# WinRM

WinRM (Windows Remote Management) is Microsoft's implementation of the WS-Management protocol, listening on TCP 5985 (HTTP) and 5986 (HTTPS), and it is the transport behind PowerShell Remoting. It is a prime lateral-movement and remote-execution target on Windows networks: with valid credentials (or an NT hash, since WinRM supports NTLM authentication and therefore pass-the-hash) for a member of the Remote Management Users group or a local admin, an attacker gets an interactive PowerShell session on the host, usually with administrative privileges. The surface is enumerating the service, obtaining and using credentials or hashes, and the command execution itself.

```bash
nmap -p5985,5986 -sV <target>
nxc winrm <target> -u user -p 'pass'           # validates creds and flags Pwn3d! (admin)
```

## Subtopics

- **[Enumeration](enumeration.md)**: detecting WinRM and its configuration.
- **[Authentication](authentication.md)**: passwords, NTLM, and pass-the-hash.
- **[Command execution](command-execution.md)**: obtaining a shell with Evil-WinRM and PowerShell Remoting.

## References

- [MS-WSMV (WS-Management)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-wsmv/)
- [HackTricks: WinRM (5985)](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
