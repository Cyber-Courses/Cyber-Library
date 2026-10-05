---
title: "Command execution: obtaining a shell over WinRM"
description: "Once authenticated, WinRM gives remote command execution through PowerShell Remoting: Evil-WinRM and native PowerShell (Enter-PSSession/Invoke-Command) open an interactive session on the host, typically as an administrator. From there an attacker runs commands, uploads tools, loads in-memory scripts, and dumps credentials, making WinRM a full foothold."
keywords:
  - command execution
  - evil-winrm
  - enter-pssession
  - invoke-command
  - powershell remoting
---

# Command execution

WinRM's purpose is remote management, so authenticated access is remote command execution. The attacker-friendly client is Evil-WinRM, which opens an interactive PowerShell session and adds conveniences (file upload/download, in-memory script and .NET assembly loading); native Windows tooling (`Enter-PSSession`, `Invoke-Command`) does the same from a Windows attack host. Because WinRM access is usually administrative, the session is a privileged foothold: run commands, stage tools, load offensive scripts in memory to avoid touching disk, and dump credentials for further lateral movement.

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

- The session is typically admin (WinRM access requires it or Remote Management Users), so command execution is a privileged foothold; confirm with `whoami /groups` and move to credential dumping and persistence.
- Evil-WinRM's in-memory load (`Invoke-Binary`, script loading) runs tooling without writing to disk, reducing artefacts; use it for offensive scripts and assemblies.
- WinRM is logged (PowerShell and WSMan event logs); it is operationally quieter than some execution methods but not invisible, so expect session and script-block logging.
- From the shell, pivot as usual: dump LSASS/SAM for more hashes (feeding more [pass-the-hash](authentication.md) to other hosts), read files, and establish persistence.

## Tools

- [Evil-WinRM](https://github.com/Hackplayers/evil-winrm)
- [PowerShell Remoting](https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands)

## References

- [HackTricks: WinRM execution](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
- [MS-WSMV](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-wsmv/)
