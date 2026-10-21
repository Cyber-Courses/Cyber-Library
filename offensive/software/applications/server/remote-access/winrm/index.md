---
title: "WinRM: attacking Windows Remote Management"
order: 8
description: "WinRM implements WS-Management on TCP 5985 (HTTP) and 5986 (HTTPS), providing remote management through PowerShell Remoting. Successful authentication can yield a shell under the account's existing privileges, subject to endpoint authorization and session constraints."
keywords:
  - winrm
  - ws-management
  - powershell remoting
  - evil-winrm
  - port 5985
---

# WinRM

WinRM (Windows Remote Management) is Microsoft's implementation of the WS-Management protocol, listening on TCP 5985 (HTTP) and 5986 (HTTPS), and it is a transport behind PowerShell Remoting. It is a useful lateral-movement and remote-execution target on Windows networks when an account is authorized for a session endpoint. Passwords and, where NTLM is enabled, NT hashes may authenticate the connection. The resulting session runs in the authenticated user's context and remains subject to operating-system permissions and endpoint restrictions; successful remoting does not itself grant administrator rights.

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
