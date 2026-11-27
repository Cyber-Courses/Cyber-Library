---
title: "Authentication: WinRM passwords, NTLM, and pass-the-hash"
order: 2
description: "WinRM supports Negotiate, Kerberos, Basic, client-certificate, and CredSSP authentication according to client, service, and endpoint configuration. Negotiate may select NTLM, which enables pass-the-hash in compatible tooling, while Kerberos use depends on valid tickets, names, and SPNs. Authentication still requires endpoint authorization."
keywords:
  - winrm authentication
  - pass-the-hash
  - ntlm
  - kerberos
  - spray
---

# Authentication

WinRM supports several configurable mechanisms: Negotiate (which selects Kerberos or NTLM), explicit Kerberos, Basic, client certificates, and CredSSP. NTLM is normally selected through Negotiate rather than exposed as a separate WSMan authentication method. Tools that implement the NTLM exchange can use an NT hash for pass-the-hash without first cracking it. Kerberos use requires a ticket valid for the account and target service, a hostname that permits SPN resolution, and compatible realm/DNS configuration; possession of an arbitrary ticket is not sufficient. Basic directly transmits a username and password in the request, is disabled by default on the WinRM service, and should be used only with TLS. Authentication is followed by authorization against the selected management resource or PowerShell session endpoint, whose security descriptor may be customized; no single group list describes every deployment.

```bash
# password and, when Negotiate selects NTLM, NT hash authentication
nxc winrm <target> -u user -p 'Password1'
nxc winrm <target> -u user -H <nthash>                 # pass-the-hash
evil-winrm -i <target> -u user -H <nthash>             # hash auth into a shell
evil-winrm -i <target> -u user -p 'Password1'
# Kerberos: use a matching ticket, realm, hostname, DNS, and service SPN
export KRB5CCNAME=ticket.ccache; evil-winrm -i <target-hostname> -u user -r <REALM>
```

## Exploitation notes

- When NTLM is enabled and selected, compatible clients can authenticate with a dumped NT hash without cracking it. Policy may disable NTLM or require another method.
- Authorization gates usage: default and custom WinRM resources or PowerShell endpoints can have different security descriptors. A valid credential is not proof that the account can create a session or run every command.
- Respect lockout and authorization scope when testing credentials. Treat tool labels such as `Pwn3d!` as a heuristic and verify the actual token, endpoint, and permitted operations in the session.
- Kerberos avoids NTLM, but the cache must contain a suitable ticket and the connection name must align with the target's SPN and realm configuration.

## References

- [HackTricks: WinRM authentication](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
- [Microsoft: WinRM authentication for remote connections](https://learn.microsoft.com/en-us/windows/win32/winrm/authentication-for-remote-connections)
- [Evil-WinRM](https://github.com/Hackplayers/evil-winrm)
