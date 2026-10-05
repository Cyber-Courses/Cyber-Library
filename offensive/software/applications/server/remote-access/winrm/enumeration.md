---
title: "Enumeration: detecting WinRM and its configuration"
description: "WinRM is identified by its listeners on 5985 (HTTP) and 5986 (HTTPS), which respond to a WS-Management probe. Enumeration confirms the service, whether HTTP or HTTPS is used, and which authentication methods are allowed, and a credential check reveals whether a given account can actually reach WinRM and whether it has administrative rights."
keywords:
  - winrm enumeration
  - 5985
  - 5986
  - wsman
  - auth methods
---

# Enumeration

WinRM enumeration confirms the service and its posture before attacking. The listeners are 5985 (HTTP) and 5986 (HTTPS), and they answer a WS-Management request (an HTTP POST to `/wsman`) distinctively, so a probe identifies WinRM even though it rides HTTP. Enumeration also establishes which transport is in use and which authentication methods the listener permits (Negotiate/NTLM, Kerberos, Basic, CredSSP). Finally, a credential check against WinRM shows whether a given account is actually allowed to connect (membership of Remote Management Users or admin) and whether that connection is privileged.

```bash
nmap -p5985,5986 -sV <target>
# WS-Management probe
curl -s -X POST http://<target>:5985/wsman -H 'Content-Type: application/soap+xml' | head
# validate an account and its WinRM access/privilege
nxc winrm <target> -u user -p 'pass'           # (Pwn3d! indicates admin via WinRM)
```

## Exploitation notes

- Open 5985/5986 plus a WS-Management response confirms WinRM; a `401` to the probe still confirms the service and reveals the offered authentication methods in the `WWW-Authenticate` headers.
- The credential check is the decisive enumeration: `nxc winrm` reports whether the account can use WinRM at all (not every valid account can) and whether it lands as admin.
- HTTPS (5986) wraps the same protocol in TLS; it does not change the attack, only the transport, though it blocks passive capture of a Basic-auth credential.
- A confirmed WinRM endpoint plus a usable credential or hash routes directly to [authentication](authentication.md) and [command execution](command-execution.md).

## References

- [HackTricks: WinRM enumeration](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
- [NetExec winrm](https://www.netexec.wiki/winrm-protocol)
