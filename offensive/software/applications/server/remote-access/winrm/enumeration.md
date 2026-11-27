---
title: "Enumeration: detecting WinRM and its configuration"
order: 1
description: "WinRM commonly listens on 5985 (HTTP) and 5986 (HTTPS), though listeners may use other ports. A WS-Management probe and its response headers can identify the service and advertised authentication schemes. Credential and privilege results remain specific to the selected resource or session endpoint."
keywords:
  - winrm enumeration
  - 5985
  - 5986
  - wsman
  - auth methods
---

# Enumeration

WinRM 2.0 commonly uses 5985 for HTTP and 5986 for HTTPS, but listeners can be configured on arbitrary ports. A WS-Management request (normally an HTTP POST to `/wsman`) and the response status and headers can identify the service and its advertised HTTP authentication schemes. A credential check establishes whether the account can access the requested WinRM resource or PowerShell session endpoint; authorization may differ between endpoints, and a tool's privilege marker is not a substitute for inspecting the resulting token and allowed operations.

```bash
nmap -p5985,5986 -sV <target>
# WS-Management probe
curl -sS -i -X POST http://<target>:5985/wsman -H 'Content-Type: application/soap+xml' | head
# test whether the account reaches the selected WinRM path; verify privileges in-session
nxc winrm <target> -u user -p 'pass'           # tool privilege labels are only heuristics
```

## Exploitation notes

- Open 5985/5986 plus a WS-Management response confirms WinRM; a `401` to the probe still confirms the service and reveals the offered authentication methods in the `WWW-Authenticate` headers.
- Credential checks are resource-specific: access to one WinRM resource or session configuration does not imply access to all endpoints. Verify privileges and permitted operations after connecting.
- HTTPS adds TLS transport protection. Over HTTP, Negotiate/Kerberos, NTLM, and CredSSP can protect ongoing messages using authentication-negotiated encryption; Basic provides no message encryption and should not be used without TLS. `AllowUnencrypted` is disabled by default.
- A confirmed WinRM endpoint plus a usable credential or hash routes directly to [authentication](authentication.md) and [command execution](command-execution.md).

## References

- [HackTricks: WinRM enumeration](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
- [Microsoft: WinRM installation and listener configuration](https://learn.microsoft.com/en-us/windows/win32/winrm/installation-and-configuration-for-windows-remote-management)
- [Microsoft: PowerShell remoting security](https://learn.microsoft.com/en-us/powershell/scripting/security/remoting/winrm-security)
- [NetExec winrm](https://www.netexec.wiki/winrm-protocol)
