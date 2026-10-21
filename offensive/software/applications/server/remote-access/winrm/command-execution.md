---
title: "Command execution: obtaining a shell over WinRM"
order: 3
description: "An account authorized for a PowerShell Remoting endpoint can execute the commands that endpoint and the account's operating-system privileges permit. Administrative impact must be established separately rather than inferred from a successful WinRM session."
keywords:
  - command execution
  - evil-winrm
  - enter-pssession
  - invoke-command
  - powershell remoting
---

# Command execution

WinRM's purpose is remote management, but authentication alone is not authorization to every PowerShell endpoint or command. Evil-WinRM can open an interactive session and add file-transfer and script-loading conveniences; native Windows tooling (`Enter-PSSession`, `Invoke-Command`) provides equivalent remoting primitives. The session runs under the authenticated user's context. Its impact depends on that account's groups, token, local access controls, and the selected session configuration; constrained or JEA endpoints may expose only a limited command set.

```bash
# interactive shell (password or hash)
evil-winrm -i <target> -u administrator -H <nthash>
#   *Evil-WinRM* PS> whoami; upload tool.exe; download C:\loot\file
#   *Evil-WinRM* PS> Invoke-Command -ScriptBlock { ... }    # in-memory
# native PowerShell remoting from a Windows host
Enter-PSSession -ComputerName <target> -Credential <cred>
Invoke-Command -ComputerName <target> -Credential <cred> -ScriptBlock { whoami }
```

## Exploitation notes

- Establish privilege explicitly with `whoami`, `whoami /groups`, and allowed-command testing. Membership in Remote Management Users permits configured remote-management access but does not make the account an administrator.
- Evil-WinRM supports in-memory script and assembly loading, but whether loaded code can perform a sensitive action still depends on the session language mode and the user's permissions.
- WinRM is logged (PowerShell and WSMan event logs); it is operationally quieter than some execution methods but not invisible, so expect session and script-block logging.
- From an administrative session, credential access and persistence may be possible; from a standard or constrained session, enumerate the accessible files, commands, and endpoint configuration without assuming those capabilities.

## Tools

- [Evil-WinRM](https://github.com/Hackplayers/evil-winrm)
- [PowerShell Remoting](https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands)
- [PowerShell Remoting security considerations](https://learn.microsoft.com/en-us/powershell/scripting/security/remoting/winrm-security)

## References

- [HackTricks: WinRM execution](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
- [MS-WSMV](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-wsmv/)
